# User Verification

You can indicate the user verification requirements during the ceremonies by setting the value in your options.

### Authenticator registration

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

### User Authentication

```php
// Public Key Credential Request Options
$publicKeyCredentialRequestOptions = new PublicKeyCredentialRequestOptions(
    random_bytes(32),
    60000, 
    'foo.example.com',
    $allowedCredentials,
    PublicKeyCredentialRequestOptions::USER_VERIFICATION_REQUIREMENT_DISCOURAGED
);
```
