# Conditional Create

{% hint style="info" %}
**New in v5.3.0**
{% endhint %}

Conditional Create allows you to register a WebAuthn credential without explicit user interaction, typically after a user has already authenticated via another method (e.g., password login). This enables a seamless upgrade path from traditional authentication to passkeys.

## How It Works

In a standard WebAuthn registration ceremony, user presence is always required (the user must interact with the authenticator). With Conditional Create (`mediation: 'conditional'`), the browser can silently create a credential after the user has already proven their identity through another means.

This is particularly useful for:

* **Passkey upgrade prompts**: After a password login, silently offer to register a passkey
* **Progressive enrollment**: Gradually migrate users from passwords to passkeys
* **Background registration**: Register credentials without interrupting the user flow

## Pure PHP Usage

There are two equivalent ways to relax the User Presence (UP) check during validation: a per-request hint on the options, or a dedicated ceremony manager. The first option is the recommended one since v5.3.0 because the hint travels with the options across the storage round-trip and applies automatically.

### Option 1 — Set `mediation` on the options (recommended)

`PublicKeyCredentialCreationOptions` exposes two constants — `MEDIATION_DEFAULT` and `MEDIATION_CONDITIONAL` — and a nullable `$mediation` property. When the property is set to `MEDIATION_CONDITIONAL`, `CheckUserWasPresent` skips the UP check at runtime regardless of which ceremony manager you use.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\PublicKeyCredentialCreationOptions;

$options = PublicKeyCredentialCreationOptions::create(
    $rpEntity,
    $userEntity,
    $challenge,
    mediation: PublicKeyCredentialCreationOptions::MEDIATION_CONDITIONAL,
);
```
{% endcode %}

{% hint style="info" %}
The `mediation` property is intentionally **not** serialized to the JSON sent to the browser — the browser receives `mediation: 'conditional'` via the JS API. The property is only used server-side and survives PHP `serialize`/`unserialize` so it can be stored in the session between the options request and the response validation.
{% endhint %}

### Option 2 — Use a dedicated ceremony manager

The `CeremonyStepManagerFactory` still provides a dedicated method that returns a manager configured with `userPresenceRequired = false`:

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\CeremonyStep\CeremonyStepManagerFactory;

$csmFactory = new CeremonyStepManagerFactory();

// Standard registration ceremony (user presence required)
$standardCeremony = $csmFactory->creationCeremony();

// Conditional create ceremony (user presence can be false)
$conditionalCeremony = $csmFactory->conditionalCreateCeremony();
```
{% endcode %}

Use the conditional ceremony manager when validating attestation responses from conditional create flows:

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\AuthenticatorAttestationResponseValidator;

// Create a validator with the conditional ceremony manager
$validator = AuthenticatorAttestationResponseValidator::create(
    ceremonyStepManager: $conditionalCeremony
);

$credentialRecord = $validator->check(
    $authenticatorAttestationResponse,
    $publicKeyCredentialCreationOptions,
    'example.com'
);
```
{% endcode %}

## Symfony Bundle Configuration

Enable conditional create per creation profile. The bundle's `CeremonyStepManagerFactory` translates the `conditional_create: true` flag into `mediation = 'conditional'` on the produced options, so existing 5.3.x configurations behave identically:

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    creation_profiles:
        passkey_upgrade:
            rp:
                id: 'example.com'
            conditional_create: true   # Enable conditional create for this profile
            authenticator_selection_criteria:
                require_resident_key: true
                user_verification: preferred
```
{% endcode %}

If you would rather decide per request — for example, only enable conditional create when the JavaScript client explicitly asks for it — opt in via the [Client Override Policy](../../symfony-bundle/advanced-behaviors/client-override-policy.md):

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    creation_profiles:
        passkey_upgrade:
            rp:
                id: 'example.com'
            client_override_policy:
                mediation:
                    enabled: true
                    allowed_values: ['default', 'conditional']
```
{% endcode %}

## Frontend Integration

On the frontend, use `mediation: 'conditional'` when calling `navigator.credentials.create()`:

{% code lineNumbers="true" %}
```javascript
const credential = await navigator.credentials.create({
    publicKey: creationOptions,
    mediation: 'conditional'
});
```
{% endcode %}

With the Stimulus controller, use the `autoRegister` option on the registration controller:

```twig
{{ stimulus_controller('@web-auth/webauthn-stimulus', {
    creationOptionsUrl: path('webauthn.controller.creation.creation.passkey_upgrade'),
    creationResultField: 'input[name="attestation"]',
    autoRegister: true
}) }}
```

{% hint style="warning" %}
Conditional Create requires browser support. Not all browsers support this feature yet. Always provide a fallback registration flow.
{% endhint %}

## See Also

* [Register Authenticators](../../pure-php/authenticator-registration.md) - Standard registration ceremony
* [Stimulus Controllers](../../symfony-ux/installation.md) - Frontend integration
* [W3C Conditional Create Explainer](https://github.com/nicovil/webauthn/wiki/Explainer:-Conditional-Create) - Specification details
