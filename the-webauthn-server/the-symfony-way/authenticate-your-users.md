# Authenticate Your Users

To authenticate your users,, you need to create a `PublicKeyCredentialRequestOptions` object. You can create this object using the .... Similarly to the authentication registration process, there is another approach.

The bundle provides a factory and manages profiles to ease the creation of the options. The factory is available as a public service: `Webauthn\Bundle\Service\PublicKeyCredentialRequestOptionsFactory`. To use it, you must first create a least one profile in your configuration file.

{% code-tabs %}
{% code-tabs-item title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    request_profiles:
        acme: ~
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="success" %}
No other option is needed to create a profile!
{% endhint %}

With this profile, now we can create options with the following code lines:

```php
use Webauthn\Bundle\Service\PublicKeyCredentialCreationOptionsFactory;
use Webauthn\PublicKeyCredentialUserEntity;

// UseEntity found using the username.
$userEntity = $userEntityRepository->findWebauthnUserByUsername('john.doe');

// Get the list of authenticators associated to the user
$credentialSources = $credentialSourceRepository->findAllForUserEntity($userEntity);

// Convert the Credential Sources into Public Key Credential Descriptors
$allowedCredentials = array_map(function (PublicKeyCredentialSource $credential) {
return $credential->getPublicKeyCredentialDescriptor();
}, $credentialSources);

$publicKeyCredentialCreationOptions = $container
    ->get(PublicKeyCredentialCreationOptionsFactory::class)
    ->create('acme', $allowedCredentials)
;
```

### Relaying Party ID

As mentioned earlier, it is preferable to indicate the Relaying Party ID. By default it is set to `null` i.e. the current domain is used.

{% code-tabs %}
{% code-tabs-item title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    request_profiles:
        acme:
            rp_id: 'example.com'
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Challenge Length

By default, the length of the challenge is 32 bytes. You may need to select a smaller or higher length. This length can be configured for each profile:

{% code-tabs %}
{% code-tabs-item title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    request_profiles:
        acme:
            challenge_length: 16
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Timeout

The default timeout is set to 60 seconds \(60 000 milliseconds\). You can change this value as follow:

{% code-tabs %}
{% code-tabs-item title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            timeout: 30000
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### User Verification

By default, the authenticator will verify the user if it is possible. You can enforce or disable the user verification using this option.

{% code-tabs %}
{% code-tabs-item title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            user_verification: !php/const Webauthn\AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_REQUIRED
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Extensions

The mechanism for generating public key credentials, as well as requesting and generating Authentication assertions, can be extended to suit particular use cases. Each case is addressed by defining a registration extension.

{% hint style="info" %}
The example below is tatolly fictive. Some extensions are [defined in the specification](https://www.w3.org/TR/webauthn/#sctn-defined-extensions) but the supports depends on the authenticators and on the relaying parties.
{% endhint %}

{% code-tabs %}
{% code-tabs-item title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            extensions:
                loc: true
                txAuthSimple: 'Please add your new authenticator'
```
{% endcode-tabs-item %}
{% endcode-tabs %}

