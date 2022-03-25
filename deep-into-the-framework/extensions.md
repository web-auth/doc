# Extensions

The mechanism for generating public key credentials, as well as requesting and generating Authentication assertions, can be extended to suit particular use cases. Each case is addressed by defining a registration extension.

{% hint style="info" %}
This library is ready to handle extension inputs and outputs, but no concrete implementations are provided.

It is up to you, depending on the extensions you want to support, to create the extension handlers.
{% endhint %}

## Creation/Request Options

The following example is totally fictive. We will add an extension input `loc=true` to the request option object.

```php
<?php

declare(strict_types=1);

use Webauthn\AuthenticationExtensions\AuthenticationExtension;
use Webauthn\AuthenticationExtensions\AuthenticationExtensionsClientInputs;
use Webauthn\PublicKeyCredentialRequestOptions;

// Extensions
$extensions = new AuthenticationExtensionsClientInputs();
$extensions->add(new AuthenticationExtension('loc', true));

// List of registered PublicKeyCredentialDescriptor classes associated to the user
$registeredPublicKeyCredentialDescriptors = …;

// Public Key Credential Request Options
$publicKeyCredentialRequestOptions = new PublicKeyCredentialRequestOptions(
    random_bytes(32),                                                           // Challenge
    60000,                                                                      // Timeout
    'foo.example.com',                                                          // Relying Party ID
    $registeredPublicKeyCredentialDescriptors,                                  // Registered PublicKeyCredentialDescriptor classes
    PublicKeyCredentialRequestOptions::USER_VERIFICATION_REQUIREMENT_PREFERRED, // User verification requirement
    $extensions
);
```

## Extension Output Checker

An Extension Output Checker will check the extension inputs and output.

It must implement the interface `Webauthn\AuthenticationExtensions\ExtensionOutputChecker` and throw an exception of type `Webauthn\AuthenticationExtension\ExtensionOutputError` in case of error.

{% hint style="info" %}
Devices may ignore the extension inputs. The extension outputs are therefore not guaranteed.
{% endhint %}

In the previous example, we asked for the location of the device and we expect to receive geolocation data in the extension output.

```php
<?php

declare(strict_types=1);

namespace Acme\Extension;

use Webauthn\AuthenticationExtensions\ExtensionOutputChecker;
use Webauthn\AuthenticationExtensions\ExtensionOutputError;

final class LocationExtensionOutputChecker
{
    public function check(AuthenticationExtensionsClientInputs $inputs, AuthenticationExtensionsClientOutputs $outputs): void
    {
        if (!$inputs->has('loc') || $inputs->get('loc') !== true) {
            return;
        }

        if (!$outputs->has('loc')) {
            //You may simply return but here we consider it is a mandatory extension output.
            throw new ExtensionOutputError(
                $inputs->get('loc'),
                'The location of the device is missing'
            );
        }

        $location = $outputs->get('loc');
        //... Proceed with the output e.g. by logging the location of the device
        // or verifying it is in a specific area.
    }
}
```

## Extension Input

To enable an authenticator feature like the geolocation, you must ask it through the creation or the request option objects.

### The Hard Way

#### Authenticator registration

```php
<?php

declare(strict_types=1);

use Webauthn\AuthenticationExtensions\AuthenticationExtension;
use Webauthn\AuthenticationExtensions\AuthenticationExtensionsClientInputs;
use Webauthn\PublicKeyCredentialCreationOptions;

// Extensions
$extensions = new AuthenticationExtensionsClientInputs();
$extensions->add(new AuthenticationExtension('loc', true));

$publicKeyCredentialCreationOptions = new PublicKeyCredentialCreationOptions(
    $rpEntity,
    $userEntity,
    $challenge,
    $publicKeyCredentialParametersList,
    $timeout,
    $excludedPublicKeyDescriptors,
    $authenticatorSelectionCriteria,
    PublicKeyCredentialCreationOptions::ATTESTATION_CONVEYANCE_PREFERENCE_NONE,
    $extensions
);
```

#### User Authentication

```php
<?php

declare(strict_types=1);

use Webauthn\AuthenticationExtensions\AuthenticationExtension;
use Webauthn\AuthenticationExtensions\AuthenticationExtensionsClientInputs;
use Webauthn\PublicKeyCredentialRequestOptions;

// Extensions
$extensions = new AuthenticationExtensionsClientInputs();
$extensions->add(new AuthenticationExtension('loc', true));

// List of registered PublicKeyCredentialDescriptor classes associated to the user
$registeredAuthenticators = $publicKeyCredentialSourceRepository->findAllForUserEntity($userEntity);
$allowedCredentials = array_map(
    static function (PublicKeyCredentialSource $credential): PublicKeyCredentialDescriptor {
        return $credential->getPublicKeyCredentialDescriptor();
    },
    $registeredAuthenticators
);

// Public Key Credential Request Options
$publicKeyCredentialRequestOptions = new PublicKeyCredentialRequestOptions(
    random_bytes(32),                                                           // Challenge
    60000,                                                                      // Timeout
    'foo.example.com',                                                          // Relying Party ID
    $allowedCredentials,                                                        // Registered PublicKeyCredentialDescriptor classes
    PublicKeyCredentialRequestOptions::USER_VERIFICATION_REQUIREMENT_PREFERRED, // User verification requirement
    $extensions                                                                 // Extensions
);
```

### The Symfony Way

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
            extensions:
                loc: true
    request_profiles:
        default:
            rp_id: 'example.com'
            extensions:
                loc: true
```
{% endcode %}
