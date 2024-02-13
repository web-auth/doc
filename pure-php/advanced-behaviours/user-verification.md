# User Verification

You can indicate the user verification requirements during the ceremonies by setting the value in your options.

### Authenticator registration

```php
$authenticatorSelectionCriteria = AuthenticatorSelectionCriteria::create()
    ->setUserVerification(AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_DISCOURAGED)
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

### User Authentication

```php
// Public Key Credential Request Options
$publicKeyCredentialRequestOptions = PublicKeyCredentialRequestOptions::create(random_bytes(32))
    ->setUserVerification(
        PublicKeyCredentialRequestOptions::USER_VERIFICATION_REQUIREMENT_DISCOURAGED
    )
;
```
