# Options Helpers (Custom Controllers)

{% hint style="info" %}
**New in v5.4.0**
{% endhint %}

The bundle ships an autowired helper that lets you write your own attestation/assertion *options* controllers without touching a single line of bundle configuration. This is the recommended path going forward: the controllers shipped with the bundle (`AttestationRequestController` / `AssertionRequestController`, driven by `creation_profiles` / `request_profiles`) stay available for backwards compatibility but the helper is more flexible and clearer.

## The entry point

`Webauthn\Bundle\Service\WebauthnOptionsResponse` is autowired and public. It exposes two factory methods, one per ceremony, returning a fluent builder:

```php
$this->options->forCreation(string $rpId, PublicKeyCredentialUserEntity|UserEntityGuesser $user): WebauthnCreationOptionsBuilder
$this->options->forRequest(string $rpId): WebauthnRequestOptionsBuilder
```

The required pieces are constructor arguments of the factory call. Everything else has a sensible default and is fluently overridable via `with…()` setters on the returned builder. The terminal call `->build($request)` returns a `JsonResponse` ready to ship.

### User: entity or guesser

For **registration** (`forCreation`), the user is required and must be passed positionally. It accepts either:

* a `PublicKeyCredentialUserEntity`, when your controller already has it in hand (typical for authenticated flows where you just read `Security::getUser()` and map it to a user entity yourself);
* a `UserEntityGuesser`, when the user has to be resolved from the request body (typical for the *new user* registration flow where you read `username` / `displayName` from the JSON payload).

For **assertion** (`forRequest`), the user is optional. Omit it for a userless ceremony (passkeys discoverable on the platform side); attach a known user via `withUser()` for a step-up / explicit login flow.

### Defaults

The creation builder applies a baseline `pubKeyCredParams` list (in priority order):

* ES256, RS256, EdDSA, ES384, ES512, PS256, RS384, RS512

`challengeLength` defaults to 32 bytes. `timeout`, `userVerification`, `attestation` are left to the W3C-recommended defaults (the user agent decides). On the request side, `allowCredentials` is automatically derived from `CredentialRecordRepositoryInterface::findAllForUserEntity()` when a user is resolved.

## Minimal controllers

### Registration

{% code title="src/Controller/Webauthn/RegisterOptionsController.php" lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

namespace App\Controller\Webauthn;

use App\Security\NewUserEntityGuesser;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\Attribute\Route;
use Webauthn\Bundle\Service\WebauthnOptionsResponse;

final class RegisterOptionsController
{
    public function __construct(
        private readonly WebauthnOptionsResponse $options,
        private readonly NewUserEntityGuesser $guesser,
    ) {
    }

    #[Route('/webauthn/register/options', methods: ['POST'])]
    public function __invoke(Request $request): JsonResponse
    {
        return $this->options
            ->forCreation('example.com', $this->guesser)
            ->build($request);
    }
}
```
{% endcode %}

`NewUserEntityGuesser` is a class **you write** that implements `Webauthn\Bundle\Security\Guesser\UserEntityGuesser`. It reads the username from the body, generates a user handle, and returns a `PublicKeyCredentialUserEntity`. The bundle does not ship a generic implementation: that logic is application-specific.

### Authentication, userless (passkeys)

{% code title="src/Controller/Webauthn/LoginOptionsController.php" lineNumbers="true" %}
```php
#[Route('/webauthn/login/options', methods: ['POST'])]
public function __invoke(Request $request): JsonResponse
{
    return $this->options
        ->forRequest('example.com')
        ->build($request);
}
```
{% endcode %}

No guesser, no allow-list. The user agent will offer the user any platform-side discoverable passkey for `example.com`.

### Authentication, user already known

{% code lineNumbers="true" %}
```php
#[Route('/webauthn/step-up/options', methods: ['POST'])]
public function __invoke(Request $request): JsonResponse
{
    $user = $this->mapToWebauthnUserEntity($this->getUser());

    return $this->options
        ->forRequest('example.com')
        ->withUser($user)
        ->build($request);
}
```
{% endcode %}

`allowCredentials` is automatically derived from your `CredentialRecordRepositoryInterface` for the resolved user. If you have a `UserEntityGuesser` instead of a ready-made entity, pass it the same way: `withUser($myGuesser)`.

## Configuring the ceremony

Every optional field has a `with…()` setter on the returned builder. Each call returns a clone, so the entry-point service stays safe to use across requests.

### Common (both creation and request)

| Setter | Default |
| --- | --- |
| `withChallengeLength(int)` | 32 |
| `withTimeout(?int)` | `null` (UA decides) |
| `withAttestation(?string)` | `null` (= `none`) |
| `withAttestationFormats(array)` | empty (any format accepted) |
| `withExtensions(AuthenticationExtensions)` | none |
| `withHints(array)` | none |
| `withClientOverrides(ClientOverridePolicy)` | none (no client field has effect) |
| `withOptionsStorage(OptionsStorage)` | the global `webauthn.options_storage` |
| `withCredentialRepository(CredentialRecordRepositoryInterface)` | the global `webauthn.credential_repository` |

`withOptionsStorage(...)` and `withCredentialRepository(...)` override the challenge-storage backend and the credential lookup repository for this options build only. Multi-tenant setups can pre-build a per-tenant builder once and reuse it across requests:

```php
return $this->options
    ->forCreation('example.com', $this->guesser)
    ->withOptionsStorage($tenantA->optionsStorage)
    ->withCredentialRepository($tenantA->credentialRepository)
    ->build($request);
```

### Creation only

| Setter | Default |
| --- | --- |
| `withAuthenticatorSelectionCriteria(AuthenticatorSelectionCriteria)` | none |
| `withPubKeyCredParams(array)` | the W3C-recommended baseline list |
| `withMediation(?string)` | `null` (= `default`); use `'conditional'` for [Conditional Create](../pure-php/advanced-behaviours/conditional-create.md) |
| `withHideExistingCredentials(bool = true)` | `false` (excluded credentials are derived from the repository) |

### Request only

| Setter | Default |
| --- | --- |
| `withUser(PublicKeyCredentialUserEntity\|UserEntityGuesser)` | none (userless ceremony) |
| `withUserVerification(?string)` | `null` (= `preferred` per W3C) |
| `withUiMode(?string)` | `null` (= `auto`); use `'immediate'` for the L3 immediate flow |
| `withAllowCredentials(array)` | derived automatically when a user is resolved |
| `withDeriveAllowCredentialsFromUser(bool = true)` | `true` |
| `withFakeCredentialGenerator(?FakeCredentialGenerator)` | the autowired generator (`SimpleFakeCredentialGenerator`); pass `null` to opt out |

#### Username-enumeration protection

When the JSON body carries a `username` (typical login form post: `{"username":"alice"}`) but `alice` does not resolve to a known user, the request builder consults the autowired `FakeCredentialGenerator` and emits **fake** `allowCredentials` instead of an empty list. From the client's standpoint, the response shape is identical whether or not the username matches a real account, so an attacker cannot probe for valid usernames.

Default behaviour (no setup needed):

```php
return $this->options
    ->forRequest('example.com')
    ->build($request);
```

Opt out (response will be a true userless / empty `allowCredentials` ceremony when the username is unknown):

```php
return $this->options
    ->forRequest('example.com')
    ->withFakeCredentialGenerator(null)
    ->build($request);
```

Or swap with a custom implementation of `Webauthn\FakeCredentialGenerator` (e.g. a hash-keyed deterministic generator backed by your `kernel.secret`) by passing it to `withFakeCredentialGenerator($custom)`.

## Client overrides

By default, **anything in the client request body is ignored**: the server alone decides every field. To let the client influence specific fields, attach a `Webauthn\Bundle\Policy\ClientOverridePolicy`. Two equivalent build paths are supported as first-class APIs.

### Typed factory (recommended)

Build the policy with named arguments and `ClientOverrideRule` value objects:

{% code lineNumbers="true" %}
```php
use Webauthn\Bundle\Policy\ClientOverridePolicy;
use Webauthn\Bundle\Policy\ClientOverrideRule;

return $this->options
    ->forRequest('example.com')
    ->withUser($user)
    ->withUserVerification(AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED)
    ->withClientOverrides(ClientOverridePolicy::fromRules(
        userVerification: ClientOverrideRule::restrictTo(['preferred', 'required']),
        extensions:       ClientOverrideRule::any(),
    ))
    ->build($request);
```
{% endcode %}

* `ClientOverrideRule::any()` — accepts any value the client submits
* `ClientOverrideRule::restrictTo($allowedValues)` — restricts the client value to a list
* Pass `null` (the default) for fields the client must NOT be able to override

### Nested-array form

The legacy `array<string, array{enabled, allowed_values?}>` shape stays supported as a first-class API; it can be useful when the policy is loaded from a config file or a database row:

{% code lineNumbers="true" %}
```php
return $this->options
    ->forRequest('example.com')
    ->withUser($user)
    ->withUserVerification(AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED)
    ->withClientOverrides(new ClientOverridePolicy([
        'user_verification' => [
            'enabled' => true,
            'allowed_values' => ['preferred', 'required'],
        ],
    ]))
    ->build($request);
```
{% endcode %}

The constructor also accepts a mix of typed `ClientOverrideRule` entries and legacy `{enabled, allowed_values?}` arrays in the same call.

### Behaviour

If the client posts `{"userVerification": "required"}`, the merged options carry `required`. If it posts `"discouraged"`, the policy rejects it (not in the allow-list) and the default (`preferred`) wins. Anything outside the policy keys is ignored regardless of what the body contains.

See [Client Override Policy](advanced-behaviors/client-override-policy.md) for the full list of overridable fields.

## Storage

The helper persists the produced options in the bundle's `OptionsStorage` automatically before returning the response. Forgetting to store options is a frequent source of *"challenge mismatch"* errors when the response controller validates the assertion later: the helper removes that risk.

If you want to bypass the helper's storage and call `OptionsStorage` yourself (e.g. to attach extra metadata), you can build the options classes directly using the lib API and skip the helper entirely.

## Why a single entry point?

`WebauthnOptionsResponse` is the only service you inject in your controller. The two builders it returns are typed (`WebauthnCreationOptionsBuilder` and `WebauthnRequestOptionsBuilder`), so the IDE autocomplete only proposes the setters that make sense for the ceremony at hand.

```php
$this->options->forCreation(...)->withMediation(...);    // ✅ creation has mediation
$this->options->forRequest(...)->withMediation(...);     // ❌ does not exist on request
$this->options->forRequest(...)->withUiMode(...);        // ✅ request has uiMode
$this->options->forCreation(...)->withUiMode(...);       // ❌ does not exist on creation
```

## Migration from `creation_profiles` / `request_profiles`

If you are still using the bundle's profile-driven controllers, the migration path is straightforward: write a controller of your own that calls the helper, then drop the matching `webauthn.controllers.creation.<name>` / `webauthn.controllers.request.<name>` config block. The route name(s) you previously had can be re-bound to your controller via the `#[Route]` attribute.

## See Also

* [Client Override Policy](advanced-behaviors/client-override-policy.md): what the client can request when a policy is set
* [Conditional Create](../pure-php/advanced-behaviours/conditional-create.md): the `withMediation('conditional')` flow
* [Signal API](../pure-php/advanced-behaviours/signal-api.md): same *helpers > config* pattern, applied to the post-ceremony signals
