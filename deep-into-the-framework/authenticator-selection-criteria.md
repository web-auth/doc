# Authenticator Selection Criteria

By default, any type of authenticator can be used by your users and interact with you application. In certain circumstances, you may need to select specific authenticators e.g. when user verification is required.

The Webauthn API and this library allow you to define a set of options to disallow the registration of authenticators that do not fulfill with the conditions.

The class `Webauthn\AuthenticatorSelectionCriteria` is designed for this purpose. It is used when generating the `Webauthn\PublicKeyCredentialCreationOptions` object.

## Available Criteria

### Authenticator Attachment Modality

You can indicate if the authenticator must be attached to the client \(platform authenticator i.e. it is usually not removable from the client device\) or must be detached \(roaming authenticator\).

Possible values are:

* `AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_NO_PREFERENCE`: there is no requirement \(default value\),
* `AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_PLATFORM`: the authenticator must be attached,
* `AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_CROSS_PLATFORM`: must be a roaming authenticator.

A primary use case for platform authenticators is to register a particular client device as a "trusted device" for future authentication. This gives the user the convenience benefit of not needing a roaming authenticator, e.g., the user will not have to dig around in their pocket for their key fob or phone.

### Resident Key

When this criterion is set to `true`, a Public Key Credential Source will be stored in the authenticator, client or client device. Such storage requires an authenticator capable to store such a resident credential.

This criterion is needed if you want to [authenticate users without username](authentication-without-username.md).

### User Verification

[Please refer to this page](user-verification.md).

## The Easy Way

```php
<?php

use Webauthn\AuthenticatorSelectionCriteria;
use Webauthn\PublicKeyCredentialCreationOptions;

$authenticatorSelectionCriteria = new AuthenticatorSelectionCriteria(
    AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_PLATFORM,     // Platform authenticator
    true,                                                                  // Resident key required
    AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_REQUIRED // User verification required
);

$publicKeyCredentialCreationOptions = $server->generatePublicKeyCredentialCreationOptions(
    $userEntity,
    PublicKeyCredentialCreationOptions::ATTESTATION_CONVEYANCE_PREFERENCE_NONE,
    $excludedPublicKeyDescriptors,
    $authenticatorSelectionCriteria
);
```

### The Hard Way

```php
$authenticatorSelectionCriteria = new AuthenticatorSelectionCriteria(
    null,
    false,
    AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_DISCOURAGED
);

$publicKeyCredentialCreationOptions = new PublicKeyCredentialCreationOptions(
    $rpEntity,
    $userEntity,
    $challenge,
    $publicKeyCredentialParametersList,
    $timeout,
    $excludedPublicKeyDescriptors,
    $authenticatorSelectionCriteria
);
```

### The Symfony Way

{% code title="config/packages/webauthn.yaml" %}
```yaml
webauthn:
    …
    creation_profiles:
        default:
            rp:
                name: 'My Application'
                id: 'example.com'
            authenticator_selection_criteria:
                attachment_mode: !php/const Webauthn\AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_NO_PREFERENCE
                require_resident_key: false
                user_verification: !php/const Webauthn\AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED
```
{% endcode %}

{% hint style="info" %}
To be written
{% endhint %}

