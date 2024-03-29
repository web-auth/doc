# Extensions

## Extension Output Checker

An Extension Output Checker will check the extension output.

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
