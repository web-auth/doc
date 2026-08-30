---
description: Upgrade guide from 5.3 to 5.4
---

# From 5.3 to 5.4

{% hint style="success" %}
No backward compatibility break. The YAML sections deprecated below keep working until 6.0.0.
{% endhint %}

Version 5.4 is the preparation step for 6.0: the profile-driven YAML configuration of the Symfony bundle is superseded by autowired helpers called from controllers you write yourself. The whole configuration surface that goes away in 6.0 is deprecated here, so you can migrate progressively while staying on the 5.x line.

## Database schema: the new `rpId` column

{% hint style="warning" %}
**Action required if you persist your credentials with Doctrine**
{% endhint %}

Version 5.4 adds the OPTIONAL `rpId` member of the credential record structure (see [w3c/webauthn#2258](https://github.com/w3c/webauthn/pull/2258)). It stores the Relying Party ID the credential was scoped to during the registration ceremony, which is useful to the Relying Parties adopting Related Origin Requests.

The column is nullable, so the existing rows are left untouched:

```sql
ALTER TABLE my_credential ADD rp_id VARCHAR(255) DEFAULT NULL;
```

Or, with the Doctrine Migrations bundle:

```bash
bin/console doctrine:migrations:diff
bin/console doctrine:migrations:migrate
```

The records created before 5.4, and those created from creation options without an explicit `rp.id`, keep `rpId` set to `null`. The serialized payloads without the `rpId` key are still accepted, and the member is omitted from the serialized output when it is `null`.

## Deprecations

### Profile-driven configuration superseded by helpers

{% hint style="warning" %}
**Deprecated in v5.4.0**
{% endhint %}

The profile- and controller-driven YAML sections are superseded by the autowired helpers introduced in 5.4. Every section keeps working until 6.0; this section lists the migration path for each. See [Options Helpers](../symfony-bundle/options-helpers.md) and [Verification Helpers](../symfony-bundle/verification-helpers.md) for the complete reference.

#### `webauthn.creation_profiles` / `webauthn.request_profiles`

Move to the `WebauthnOptionsResponse` helper from a controller of your own.

```yaml
# Before (deprecated)
webauthn:
    creation_profiles:
        default:
            rp:
                id: 'example.com'
            challenge_length: 32
            authenticator_selection_criteria:
                user_verification: 'preferred'
                resident_key: 'preferred'
            attestation_conveyance: 'none'
    request_profiles:
        default:
            rp_id: 'example.com'
            user_verification: 'preferred'
```

```php
// After
use Webauthn\AuthenticatorSelectionCriteria;
use Webauthn\Bundle\Service\WebauthnOptionsResponse;

#[Route('/webauthn/register/options', methods: ['POST'])]
public function __invoke(Request $request): JsonResponse
{
    return $this->options
        ->forCreation('example.com', $this->newUserGuesser)
        ->withChallengeLength(32)
        ->withAuthenticatorSelectionCriteria(
            AuthenticatorSelectionCriteria::create(
                AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_NO_PREFERENCE,
                AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED,
                AuthenticatorSelectionCriteria::RESIDENT_KEY_REQUIREMENT_PREFERRED,
            )
        )
        ->build($request);
}
```

#### `webauthn.controllers`

Replace each `controllers.creation[name]` and `controllers.request[name]` block by a pair of user controllers, one for the options endpoint and one for the response endpoint. The routes move to `#[Route]` attributes; the per-controller `host`, `allowed_origins`, `allow_subdomains`, `hide_existing_credentials`, `user_entity_guesser` and the other options all have a direct `with*()` equivalent on the helpers.

The full refactored example is on the [Verification Helpers](../symfony-bundle/verification-helpers.md) page.

#### `webauthn.client_override_policy`

Build a `ClientOverridePolicy` inline in the controller and attach it to the helper. 5.4 ships a typed `ClientOverrideRule` value object that makes the call site much clearer than the legacy nested-array form (both shapes stay supported as first-class APIs):

```yaml
# Before (deprecated)
webauthn:
    client_override_policy:
        user_verification:
            enabled: true
            allowed_values: [preferred, required]
```

```php
// After (5.4), typed factory recommended
use Webauthn\Bundle\Policy\ClientOverridePolicy;
use Webauthn\Bundle\Policy\ClientOverrideRule;

return $this->options
    ->forRequest('example.com')
    ->withClientOverrides(ClientOverridePolicy::fromRules(
        userVerification: ClientOverrideRule::restrictTo(['preferred', 'required']),
    ))
    ->build($request);
```

```php
// Or the legacy nested-array form, also first-class
return $this->options
    ->forRequest('example.com')
    ->withClientOverrides(new ClientOverridePolicy([
        'user_verification' => [
            'enabled' => true,
            'allowed_values' => ['preferred', 'required'],
        ],
    ]))
    ->build($request);
```

See [Client Override Policy](../symfony-bundle/advanced-behaviors/client-override-policy.md).

#### `webauthn.allowed_origins` / `webauthn.allow_subdomains`

Two migration paths, depending on your topology.

**Single-origin app (most common)**: drop the YAML node entirely. The verifier falls back to the W3C-recommended same-origin check against the request host. Nothing else to do.

**Multi-origin app**: inject a Symfony parameter into the helper.

```yaml
# Before (deprecated)
webauthn:
    allowed_origins:
        - 'https://app.example.com'
        - 'https://admin.example.com'
    allow_subdomains: true
```

```yaml
# After, in config/services.yaml
parameters:
    app.webauthn.origins:
        - 'https://app.example.com'
        - 'https://admin.example.com'
```

```php
// After, in every verification controller
use Symfony\Component\DependencyInjection\Attribute\Autowire;

public function __construct(
    private readonly WebauthnResponseVerifier $verifier,
    #[Autowire('%app.webauthn.origins%')]
    private readonly array $origins,
) {}

#[Route('/webauthn/register', methods: ['POST'])]
public function __invoke(Request $request): Response
{
    $result = $this->verifier
        ->forAttestation('example.com')
        ->withAllowedOrigins($this->origins)
        ->withAllowSubdomains(true)
        ->verify($request);
    // ...
}
```

The list lives in a single Symfony parameter, the security-critical decision is visible right where it applies, and the per-route overrides become trivial.

### Reserved-For-Future-Use accessors

{% hint style="warning" %}
**Deprecated in v5.4.0**
{% endhint %}

The RFU1 and RFU2 bits of the authenticator data flags are reserved by the specification and are set to zero by the authenticators, so reading them back brings nothing to a Relying Party. The accessors exposing them are removed in 6.0.0:

* `Webauthn\AuthenticatorData::getReservedForFutureUse1()` and `getReservedForFutureUse2()`
* `Webauthn\Bundle\Security\Authentication\Token\WebauthnToken::getReservedForFutureUse1()` and `getReservedForFutureUse2()`
* the `$reservedForFutureUse1` and `$reservedForFutureUse2` arguments of the `WebauthnToken` constructor

Nothing changes at runtime in 5.4: the constructor signature, the stored values and the serialized payload are untouched, and a deprecation notice is emitted only when a getter is called. Drop the calls, there is no replacement. The raw flags byte stays available:

```php
$flags = ord($authenticatorData->flags);
```

Use `isBackupEligible()` and `isBackedUp()` to read the backup flags.

{% hint style="danger" %}
In 6.0.0 the serialized payload of `WebauthnToken` loses two entries, so a session created by a 5.x application cannot be unserialized by a 6.0 one. Plan to invalidate the existing sessions when you upgrade.
{% endhint %}

### PseudoRandomFunctionInputExtensionBuilder::requiresHmacSecretMc

{% hint style="warning" %}
**Deprecated in v5.4.0**
{% endhint %}

The method named the CTAP `hmac-secret-mc` extension in an API that is abstract over the authenticator implementation. Use `requiresMultipleCredentialEvaluation()`, which describes the input shape the method actually detects. The old name delegates to the new one until 6.0.0. See the [PRF Extension](../pure-php/advanced-behaviours/prf-extension.md) page.

## Other changes

Opt-in features, nothing to do on your side unless you want them:

* **Ceremony origin pinning**: the options request records its own origin next to the challenge, and the verifier can require the response to be produced on that very origin, on top of the allow list. Enable it with `withCeremonyOriginPinning()` or the `webauthn.ceremony_origin_pinning` node. It fails closed on a ceremony stored without an origin, which is why it defaults to `false`. See [Verification Helpers](../symfony-bundle/verification-helpers.md).
* **Related Origin Requests**: the `/.well-known/webauthn` endpoint warns when the published origins resolve to more than five distinct eTLD+1 labels, because the clients silently ignore the extra ones. Configure it under `webauthn.related_origins`.
* **Signal API helpers**: a factory and a JSON envelope for the WebAuthn Level 3 signal methods. See [Signal API](../pure-php/advanced-behaviours/signal-api.md).
* **CTAP 2.1 authenticator extensions**: `credProtect`, `credBlob`, `getCredBlob` and `minPinLength`. See [Authenticator Extensions](../pure-php/advanced-behaviours/authenticator-extensions.md).
* **Fully-specified COSE algorithm identifiers** from RFC 9864 are accepted.
* **Immediate mediation** is supported on `navigator.credentials.get()`, and the Stimulus controllers use the native `parseCreationOptionsFromJSON`, `parseRequestOptionsFromJSON` and `toJSON` helpers along with `PublicKeyCredential.getClientCapabilities()`.
