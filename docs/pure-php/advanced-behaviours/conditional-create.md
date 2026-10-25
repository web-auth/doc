# Conditional create

!!! info

    **New in v5.3.0**

Conditional Create allows you to register a WebAuthn credential without explicit user interaction, typically after a user has already authenticated via another method (e.g., password login). This enables a seamless upgrade path from traditional authentication to passkeys.

## How it works

In a standard WebAuthn registration ceremony, user presence is always required (the user must interact with the authenticator). With Conditional Create (`mediation: 'conditional'`), the browser can silently create a credential after the user has already proven their identity through another means.

Because the credential is created without any interaction, the User Present (UP) and User Verified (UV) flags of the authenticator data may both be `false`. The validation relaxes the two checks accordingly.

!!! info

    **Changed in v5.3.6:** the User Verification check is relaxed as well. Until then only the User Presence check was, so a conditional registration made against a profile requiring user verification was rejected with `User authentication required.`.

This is particularly useful for:

* **Passkey upgrade prompts**: After a password login, silently offer to register a passkey
* **Progressive enrollment**: Gradually migrate users from passwords to passkeys
* **Background registration**: Register credentials without interrupting the user flow

## Pure PHP usage

There are two equivalent ways to relax the User Presence (UP) and User Verification (UV) checks during validation: a per-request hint on the options, or a dedicated ceremony manager. The first option is the recommended one since v5.3.0 because the hint travels with the options across the storage round-trip and applies automatically.

### Option 1 — set `mediation` on the options (recommended)

`PublicKeyCredentialCreationOptions` exposes two constants, `MEDIATION_DEFAULT` and `MEDIATION_CONDITIONAL`, and a nullable `$mediation` property. When the property is set to `MEDIATION_CONDITIONAL`, `CheckUserWasPresent` skips the UP check and `CheckUserVerification` skips the UV check at runtime, regardless of which ceremony manager you use.

```php linenums="1"
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

!!! info

    The `mediation` property is intentionally **not** serialized to the JSON sent to the browser — the browser receives `mediation: 'conditional'` via the JS API. The property is only used server-side and survives PHP `serialize`/`unserialize` so it can be stored in the session between the options request and the response validation.

### Option 2 — use a dedicated ceremony manager

The `CeremonyStepManagerFactory` still provides a dedicated method that returns a manager where neither the User Presence nor the User Verification check is enforced:

```php linenums="1"
<?php

declare(strict_types=1);

use Webauthn\CeremonyStep\CeremonyStepManagerFactory;

$csmFactory = new CeremonyStepManagerFactory();

// Standard registration ceremony (user presence required)
$standardCeremony = $csmFactory->creationCeremony();

// Conditional create ceremony (user presence can be false)
$conditionalCeremony = $csmFactory->conditionalCreateCeremony();
```

Use the conditional ceremony manager when validating attestation responses from conditional create flows:

```php linenums="1"
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

## Symfony bundle configuration

Enable conditional create per creation profile. The bundle's `CeremonyStepManagerFactory` translates the `conditional_create: true` flag into `mediation = 'conditional'` on the produced options, so existing 5.3.x configurations behave identically:

```yaml title="config/packages/webauthn.yaml" linenums="1"
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

If you would rather decide per request — for example, only enable conditional create when the JavaScript client explicitly asks for it — opt in via the [Client Override Policy](../../symfony-bundle/advanced-behaviors/client-override-policy.md):

```yaml title="config/packages/webauthn.yaml" linenums="1"
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

## Frontend integration

On the frontend, use `mediation: 'conditional'` when calling `navigator.credentials.create()`:

```javascript linenums="1"
const credential = await navigator.credentials.create({
    publicKey: creationOptions,
    mediation: 'conditional'
});
```

With the Stimulus controller, use the `autoRegister` option on the registration controller:

```twig
{{ stimulus_controller('@web-auth/webauthn-stimulus', {
    creationOptionsUrl: path('webauthn.controller.creation.creation.passkey_upgrade'),
    creationResultField: 'input[name="attestation"]',
    autoRegister: true
}) }}
```

!!! warning

    Conditional Create requires browser support. Not all browsers support this feature yet. Always provide a fallback registration flow.

## See also

* [Register Authenticators](../../pure-php/authenticator-registration.md) - Standard registration ceremony
* [Stimulus Controllers](../../symfony-ux/installation.md) - Frontend integration
* [W3C Conditional Create Explainer](https://github.com/nicovil/webauthn/wiki/Explainer:-Conditional-Create) - Specification details
