---
description: Lucky Symfony applications!
---

# The Symfony Way

An official bundle is provided in the package `web-auth/webauthn-symfony-bundle`.

{% hint style="success" %}
Starting at v3.2.4, the bundle can be installed on Symfony 4.4 or 5.0+.
{% endhint %}

{% hint style="info" %}
If you use Laravel, you may be interested in [this project: https://github.com/asbiin/laravel-webauthn](https://github.com/asbiin/laravel-webauthn)
{% endhint %}

Before installing it, please make sure you installed and configured:

* The package [`symfony/psr-http-message-bridge`](https://symfony.com/doc/current/components/psr7.html) ,
* The package [`nyholm/psr7`](https://github.com/Nyholm/psr7) or  any [other PSR-7](https://packagist.org/providers/psr/http-factory-implementation) package,
* The [SensioFremaworkExtraBundle](https://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/index.html) and enabled the PSR-7 support.

{% code title="config/packages/sensio\_framework\_extra.yaml" %}
```yaml
sensio_framework_extra:
    psr_message:
        enabled: true
```
{% endcode %}

If you are using Symfony Flex then the bundle will automatically be installed and the default configuration will be set. Otherwise you need to add it in your `AppKernel.php` file:

{% code title="src/AppKernel.php" %}
```php
<?php

public function registerBundles()
{
    $bundles = [
        // ...
        new Webauthn\Bundle\WebauthnBundle(),
    ];
}And add the Webauthn Route Loader:
```
{% endcode %}

{% code title="config/routes/webauthn\_routes.php" %}
```php
<?php

declare(strict_types=1);

use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->import('.', 'webauthn');
};
```
{% endcode %}

## Repositories

The first step is to create [your credential](../../pre-requisites/credential-source-repository.md) and [user entity repositories](../../pre-requisites/user-entity-repository.md).

Only [Doctrine ORM based repositories are provided](entities-with-doctrine.md). Other storage systems like filesystem or Doctrine ODM may be added in the future but, at the moment, you have to create these from scratch.

## Configuration

With Flex, you have a minimal configuration file installed through a Flex Recipe. You must set the repositories you have just created. You also have to modify the environment variables `RELAYING_PARTY_ID` and `RELAYING_PARTY_NAME`.

You may also need to adjust other parameters.

If you don’t use Flex, hereafter an example of configuration file:

{% code title="config/packages/webauthn.yaml" %}
```yaml
webauthn:
#    logger: null # PSR-3 compatible logging service
    credential_repository: 'Webauthn\Bundle\Repository\DummyPublicKeyCredentialSourceRepository' # CREATE YOUR REPOSITORY AND CHANGE THIS!
    user_repository: 'Webauthn\Bundle\Repository\DummyPublicKeyCredentialUserEntityRepository' # CREATE YOUR REPOSITORY AND CHANGE THIS!
    token_binding_support_handler: 'Webauthn\TokenBinding\IgnoreTokenBindingHandler' # We ignore the token binding instructions by default
    creation_profiles: # Authenticator registration profiles
        default: # Unique name of the profile
            rp: # Relaying Party information
                name: '%env(RELAYING_PARTY_NAME)%' # CHANGE THIS! or create the corresponding env variable
                id: '%env(RELAYING_PARTY_ID)%' # Please adapt the env file with the correct relaying party ID or set null
#                icon: null # Secured image (data:// scheme)
#            challenge_length: 32
#            timeout: 60000
#            authenticator_selection_criteria:
#                attachment_mode: !php/const Webauthn\AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_NO_PREFERENCE
#                require_resident_key: false
#                user_verification: !php/const Webauthn\AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED
#            extensions:
#                loc: true
#            public_key_credential_parameters: # You should not change this list
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_EdDSA #Order is important. Preferred algorithms go first
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_ES256
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_ES256K
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_ES384
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_ES512
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_RS256
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_RS384
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_RS512
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_PS256
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_PS384
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_PS512
#            attestation_conveyance: !php/const Webauthn\PublicKeyCredentialCreationOptions::ATTESTATION_CONVEYANCE_PREFERENCE_NONE
    request_profiles: # Authentication profiles
        default: # Unique name of the profile
            rp_id: '%env(RELAYING_PARTY_ID)%' # Please adapt the env file with the correct relaying party ID or set null
#            challenge_length: 32
#            timeout: 60000
#            user_verification: !php/const Webauthn\AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED
#            extensions:
#                loc: true
#    metadata_service:
#        enabled: false
#        repository: 'App\Repository\MetadataStatementRepository'
```
{% endcode %}

### Creation Profiles

{% hint style="success" %}
If you don't create the `creation_profiles` section, a `default` profile is set.
{% endhint %}

#### Relaying Party \(rp\)

The realying Party corresponds to your application. Please refer [to this page](../../pre-requisites/the-relaying-party.md) for more information.

{% hint style="warning" %}
The parameter `id` is optional but highly recommended.
{% endhint %}

#### Challenge Length

By default, the length of the challenge is 32 bytes. You may need to select a smaller or higher length. This length can be configured for each profile:

{% code title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            rp:
                name: 'ACME Webauthn Server'
            challenge_length: 16
```
{% endcode %}

#### Timeout

The default timeout is set to 60 seconds \(60 000 milliseconds\). You can change this value as follows:

{% code title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            rp:
                name: 'ACME Webauthn Server'
            timeout: 30000
```
{% endcode %}

#### Authenticator Selection Criteria

This set of options allows you to select authenticators depending on their capabilities. The values are described in [the advanced concepts](../../deep-into-the-framework/authenticator-selection-criteria.md) of the protocol.

{% code title="app/config/webauthn.yaml" %}
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
{% endcode %}

#### Public Key Credential Parameters

This option indicates the algorithms allowed for your application. By default, a large list of algorithms is defined, but you can add custom algorithms or reduce the list.

{% hint style="info" %}
The order is important. Preferred algorithms go first.
{% endhint %}

{% hint style="warning" %}
It is not recommended changing the default list unless you exactly know what you are doing.
{% endhint %}

{% code title="app/config/webauthn.yaml" %}
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
{% endcode %}

#### Attestation Conveyance

If you need the [attestation of the authenticator](../../deep-into-the-framework/attestation-and-metadata-statement.md), you can specify the preference regarding attestation conveyance during credential generation.

{% hint style="warning" %}
Please note that the metadata service is mandatory when you use this option.
{% endhint %}

{% hint style="warning" %}
The use of Attestation Statements is generally not recommended unless you REALLY need this information.
{% endhint %}

{% code title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            rp:
                name: 'ACME Webauthn Server'
            attestation_conveyance: !php/const Webauthn\PublicKeyCredentialCreationOptions::ATTESTATION_CONVEYANCE_PREFERENCE_DIRECT
```
{% endcode %}

#### Extensions

You can set as many extensions as you want in the profile. Please also [refer to this page](../../deep-into-the-framework/extensions.md) for more information.

{% hint style="info" %}
The example below is totally fictive. Some extensions are [defined in the specification](https://www.w3.org/TR/webauthn/#sctn-defined-extensions) but the support depends on the authenticators, on the browsers and on the relaying parties \(your applications\).
{% endhint %}

{% code title="app/config/webauthn.yaml" %}
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
{% endcode %}

### Request Profiles

{% hint style="success" %}
If you don't create the `creation_profiles` section, a `default` profile is set.
{% endhint %}

The parameters for the request profiles \(i.e. the authentication\) are very similar to the creation profiles. The only difference is that you don’t need all the detail of the Relaying Party, but only its ID \(i.e. its domain\).

{% code title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    request_profiles:
        acme:
            rp_id: 'example.com'
```
{% endcode %}

Please note that all parameters are optional. The following configuration is perfectly valid. However, and as mentioned above, the parameter `id` is highly recommended.

{% code title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    request_profiles:
        acme: ~
```
{% endcode %}

