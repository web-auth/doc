# Authentication without username

With Webauthn, it is possible to authenticate a user without username. This behavior implies several constraints:

1. During the registration of the authenticator, a [Resident Key must have been asked](authenticator-selection-criteria.md#resident-key),
2. The user verification is required,
3. The list of allowed authenticators must be empty

{% hint style="info" %}
In case of failure, you should continue with the standard authentication process i.e. by asking the username of the user.
{% endhint %}

## The Hard Way

Selection criteria for the registration of the authenticator:

```php
$authenticatorSelectionCriteria = new AuthenticatorSelectionCriteria(
    AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_NO_PREFERENCE,
    true,                                                                  // Resident key required
    AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_REQUIRED // User verification required
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

The Request Options:

```php
// Public Key Credential Request Options
$publicKeyCredentialRequestOptions = new PublicKeyCredentialRequestOptions(
    random_bytes(32),
    60000, 
    'foo.example.com',
    [],
    PublicKeyCredentialRequestOptions::USER_VERIFICATION_REQUIREMENT_REQUIRED
);
```
