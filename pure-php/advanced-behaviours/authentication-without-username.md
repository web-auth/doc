# Authentication without username

With Webauthn, it is possible to authenticate a user without username. This behavior implies several constraints:

1. During the registration of the authenticator, a [Resident Key must have been asked](authenticator-selection-criteria.md#resident-key),
2. The user verification is required,
3. The list of allowed authenticators must be empty

{% hint style="info" %}
In case of failure, you should continue with the standard authentication process i.e. by asking the username of the user.
{% endhint %}

Selection criteria for the registration of the authenticator:

{% code lineNumbers="true" %}
```php
use Webauthn\AuthenticatorSelectionCriteria;
use Webauthn\PublicKeyCredentialCreationOptions;


$authenticatorSelectionCriteria = AuthenticatorSelectionCriteria::create(
    userVerification: AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_REQUIRED,
    residentKey: AuthenticatorSelectionCriteria::RESIDENT_KEY_REQUIREMENT_REQUIRED,
);

$publicKeyCredentialCreationOptions = PublicKeyCredentialCreationOptions::create(
    $rpEntity,
    $userEntity,
    $challenge,
    authenticatorSelection: $authenticatorSelectionCriteria
);
```
{% endcode %}

The Request Options:

{% code lineNumbers="true" %}
```php
// Public Key Credential Request Options

use Webauthn\PublicKeyCredentialRequestOptions;

$publicKeyCredentialRequestOptions = PublicKeyCredentialRequestOptions::create(
    random_bytes(32),
    userVerification: PublicKeyCredentialRequestOptions::USER_VERIFICATION_REQUIREMENT_REQUIRED
);
```
{% endcode %}

{% hint style="success" %}
The default values for the user verification and the resident key are set to preferred and resident keys may be created if the authenticator is compatible. This means that some users may log in without username.
{% endhint %}
