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

The `CeremonyStepManagerFactory` provides a dedicated method for creating a ceremony manager that allows user presence to be false:

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

Enable conditional create per creation profile:

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
