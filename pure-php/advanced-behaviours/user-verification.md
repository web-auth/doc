# User Verification

You can indicate the user verification requirements during the ceremonies by setting the value in your options.

### Authenticator registration

{% code lineNumbers="true" %}
```php
use Webauthn\AuthenticatorSelectionCriteria;
use Webauthn\PublicKeyCredentialCreationOptions;

$publicKeyCredentialCreationOptions =
    PublicKeyCredentialCreationOptions::create(
        $rpEntity,
        $userEntity,
        $challenge,
        authenticatorSelection: AuthenticatorSelectionCriteria::create(AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_DISCOURAGED)
    )
;
```
{% endcode %}

### User Authentication

{% code lineNumbers="true" %}
```php
// Public Key Credential Request Options
use Webauthn\PublicKeyCredentialRequestOptions;

$publicKeyCredentialRequestOptions = PublicKeyCredentialRequestOptions::create(
    random_bytes(32),
    userVerification: PublicKeyCredentialRequestOptions::USER_VERIFICATION_REQUIREMENT_DISCOURAGED
);
```
{% endcode %}
