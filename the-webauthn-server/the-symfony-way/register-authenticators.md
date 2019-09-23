# Register Authenticators

As described in the previous pages, you need to create a `PublicKeyCredentialCreationOptions` object to register new authenticators. You can create this object using the .... But there is another way to do that.

The bundle provides a factory and manages profiles to ease the creation of the options. The factory is available as a public service: `Webauthn\Bundle\Service\PublicKeyCredentialCreationOptionsFactory`. To use it, you must first create a least one profile in your configuration file.

{% code-tabs %}
{% code-tabs-item title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    creation_profiles:
        acme: #Unique name of the profile
            rp: # rp stands for Relaying Party
                name: 'ACME Webauthn Server'
                id: 'acme.com'
                icon: 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABQAAAAUCAMAAAC6V+0/AAAAwFBMVEXm7NK41k3w8fDv7+q01Tyy0zqv0DeqyjOszDWnxjClxC6iwCu11z6y1DvA2WbY4rCAmSXO3JZDTxOiwC3q7tyryzTs7uSqyi6tzTCmxSukwi9aaxkWGga+3FLv8Ozh6MTT36MrMwywyVBziSC01TbT5ZW9z3Xi6Mq2y2Xu8Oioxy7f572qxzvI33Tb6KvR35ilwTmvykiwzzvV36/G2IPw8O++02+btyepyDKvzzifvSmw0TmtzTbw8PAAAADx8fEC59dUAAAA50lEQVQYV13RaXPCIBAG4FiVqlhyX5o23vfVqUq6mvD//1XZJY5T9xPzzLuwgKXKslQvZSG+6UXgCnFePtBE7e/ivXP/nRvUUl7UqNclvO3rpLqofPDAD8xiu2pOntjamqRy/RqZxs81oeVzwpCwfyA8A+8mLKFku9XfI0YnSKXnSYZ7ahSII+AwrqoMmEFKriAeVrqGM4O4Z+ADZIhjg3R6LtMpWuW0ERs5zunKVHdnnnMLNQqaUS0kyKkjE1aE98b8y9x9JYHH8aZXFMKO6JFMEvhucj3Wj0kY2D92HlHbE/9Vk77mD6srRZqmVEAZAAAAAElFTkSuQmCC'
```
{% endcode-tabs-item %}

{% code-tabs-item title=undefined %}
```

```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
The `name` is mandatory ; other options are `null` by default.
{% endhint %}

{% hint style="warning" %}
The option id is highly recommended. See [this page](../../pre-requisites/the-relaying-party.md) for acceptable values.
{% endhint %}

With this profile, now we can create options with the following code lines:

```php
use Webauthn\Bundle\Service\PublicKeyCredentialCreationOptionsFactory;
use Webauthn\PublicKeyCredentialUserEntity;

$userEntity = new PublicKeyCredentialUserEntity(
    'john.doe',
    'ea4e7b55-d8d0-4c7e-bbfa-78ca96ec574c',
    'John Doe'
);

$publicKeyCredentialCreationOptions = $container
    ->get(PublicKeyCredentialCreationOptionsFactory::class)
    ->create('acme', $userEntity)
;
```

### Challenge Length

By default, the length of the challenge is 32 bytes. You may need to select a smaller or higher length. This length can be configured for each profile:

{% code-tabs %}
{% code-tabs-item title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            rp:
                name: 'ACME Webauthn Server'
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
            rp:
                name: 'ACME Webauthn Server'
            timeout: 30000
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Authenticator Selection Criteria

This set of options allows you to select authenticators depending on their capabilities. The values are described in [the advanced concepts](../../deep-into-the-framework/authenticator-selection-criteria.md) of the protocol.

{% code-tabs %}
{% code-tabs-item title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            rp:
                name: 'ACME Webauthn Server'
            authenticator_selection_criteria:
                attachment_mode: !php/const Webauthn\AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_PLATFORM
                require_resident_key: true
                user_verification: !php/const Webauthn\AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_REQUIRED
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Public Key Credential Parameters

This option indicates the algorithms allowed for your application. By default, a large list of algorithms is defined, but you can add custom algorithms or reduce the list.

{% hint style="info" %}
The oredr is important. Preferred algorithms go first.
{% endhint %}

{% hint style="warning" %}
It is not recommended to change the default list unless you exactly know what you are doing.
{% endhint %}

{% code-tabs %}
{% code-tabs-item title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            rp:
                name: 'ACME Webauthn Server'
            public_key_credential_parameters:
                - !php/const Cose\Algorithms::COSE_ALGORITHM_ES256
                - !php/const Cose\Algorithms::COSE_ALGORITHM_RS256
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Attestation Conveyance

If you need the [attestation of the authenticator](../../deep-into-the-framework/attestation-and-metadata-statement.md), you can  specify the preference regarding attestation conveyance during credential generation.

{% hint style="warning" %}
Please note that the metadata service is mandatory to use this option.
{% endhint %}

{% hint style="danger" %}
The use of Attestation Statements is generally not recommended unless you REALLY need this information.
{% endhint %}

{% code-tabs %}
{% code-tabs-item title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            rp:
                name: 'ACME Webauthn Server'
            attestation_conveyance: !php/const Webauthn\PublicKeyCredentialCreationOptions::ATTESTATION_CONVEYANCE_PREFERENCE_DIRECT
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
            rp:
                name: 'ACME Webauthn Server'
            extensions:
                loc: true
                txAuthSimple: 'Please add your new authenticator'
```
{% endcode-tabs-item %}
{% endcode-tabs %}

