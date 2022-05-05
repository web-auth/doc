# Authenticator Selection Criteria

By default, any type of authenticator can be used by your users and interact with you application. In certain circumstances, you may need to select specific authenticators e.g. when user verification is required.

The Webauthn API and this library allow you to define a set of options to disallow the registration of authenticators that do not fulfill with the conditions.

The class `Webauthn\AuthenticatorSelectionCriteria` is designed for this purpose. It is used when generating the `Webauthn\PublicKeyCredentialCreationOptions` object.

## Available Criteria

### Authenticator Attachment Modality

You can indicate if the authenticator must be attached to the client (platform authenticator i.e. it is usually not removable from the client device) or must be detached (roaming authenticator).

Possible values are:

* `AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_NO_PREFERENCE`: there is no requirement (default value),
* `AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_PLATFORM`: the authenticator must be attached,
* `AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_CROSS_PLATFORM`: must be a roaming authenticator.

A primary use case for platform authenticators is to register a particular client device as a "trusted device" for future authentication. This gives the user the convenience benefit of not needing a roaming authenticator, e.g., the user will not have to dig around in their pocket for their key fob or phone.

### Resident Key

With this criterion, a Public Key Credential Source will be stored in the authenticator, client or client device. Such storage requires an authenticator capable to store such a resident credential.

{% hint style="info" %}
A resident key shall be created you want to [authenticate users without username](authentication-without-username.md).
{% endhint %}

### User Verification

[Please refer to this page](../pure-php/advanced-behaviours/user-verification.md).

### Example

With this example, with require the user verification (PIN, fingerprint...), a resident key and an authenticator embedded onto a device. This is typacally what you will require for Windows Hello or Face ID authentication.

```php
$authenticatorSelectionCriteria = AuthenticatorSelectionCriteria::create()
    ->setUserVerification(AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_REQUIRED)
    ->setResidentKey(AuthenticatorSelectionCriteria::RESIDENT_KEY_REQUIREMENT_REQUIRED)
    ->setAuthenticatorAttachment(AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_PLATFORM)
;

$publicKeyCredentialCreationOptions =
    PublicKeyCredentialCreationOptions::create(
        $rpEntity,
        $userEntity,
        $challenge,
        $publicKeyCredentialParametersList,
    )
    ->setAuthenticatorSelection($authenticatorSelectionCriteria)
;
```
