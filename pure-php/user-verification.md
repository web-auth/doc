# User Verification

User verification may be instigated through various authorization gesture modalities: a touch plus PIN code, password entry, or biometric recognition (presenting a fingerprint). The intent is to be able to distinguish individual users.

Eligible authenticators are filtered and only capable of satisfying this requirement will interact with the user.

Possible user verification values are:

* `required`: this value indicates that the application requires user verification for the operation and will fail the operation if the response does not have the `UV` flag set.
* `preferred`: this value indicates that the application prefers user verification for the operation if possible, but will not fail the operation if the response does not have the `UV` flag set.
* `discouraged`: this value indicates that the application does not want user verification employed during the operation (e.g.,in the interest of minimizing disruption to the user interaction flow).

Public constants are provided by `AuthenticatorSelectionCriteria`.

* `AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_REQUIRED`
* `AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED`
* `AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_DISCOURAGED`

## The Hard Way

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

## The Symfony Way

The easiest way to manage that is by using the creation and request profiles.

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
                user_verification: !php/const Webauthn\AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED
    request_profiles:
        default:
            rp_id: 'example.com'
            user_verification: !php/const Webauthn\AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED
```
{% endcode %}
