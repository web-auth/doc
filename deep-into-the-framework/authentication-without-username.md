# Authentication without username

With Webauthn, it is possible to authenticate a user without username. This behavior implies several constraints:

1. During the registration of the authenticator, a [Resident Key must have been asked](authenticator-selection-criteria.md#resident-key),
2. The user verification is required,
3. The list of allowed authenticators must be empty

{% hint style="info" %}
In case of failure, you should continue with the standard authentication process i.e. by asking the username of the user.
{% endhint %}

Selection criteria for the registration of the authenticator:

```php
$authenticatorSelectionCriteria = AuthenticatorSelectionCriteria::create()
    ->setUserVerification(AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_REQUIRED)
    ->setResidentKey(AuthenticatorSelectionCriteria::RESIDENT_KEY_REQUIREMENT_REQUIRED)
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

The Request Options:

```php
// Public Key Credential Request Options

$publicKeyCredentialRequestOptions = PublicKeyCredentialRequestOptions::create(random_bytes(32))
    ->setUserVerification(
        PublicKeyCredentialRequestOptions::USER_VERIFICATION_REQUIREMENT_REQUIRED
    )
;
```

{% hint style="success" %}
The default values for the user verification and the resident key are set to preferred and resident keys may be created if the authenticator is compatible. This means that some users may log in without username.
{% endhint %}
