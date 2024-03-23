# Extensions

## Extension Output Checker

An Extension Output Checker will check the extension output.

It must implement the interface `Webauthn\AuthenticationExtensions\ExtensionOutputChecker` and throw an exception of type `Webauthn\AuthenticationExtension\ExtensionOutputError` in case of error.

{% hint style="info" %}
Devices may ignore the extension inputs. The extension outputs are therefore not guaranteed.
{% endhint %}

In the previous example, we asked for the location of the device and we expect to receive geolocation data in the extension output.

{% code lineNumbers="true" %}
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
        if (!$inputs->has('uvm') || $inputs->get('uvm') !== true) {
            return;
        }

        if (!$outputs->has('uvm')) {
            //You may simply return but here we consider it is a mandatory extension output.
            throw new ExtensionOutputError(
                $inputs->get('uvm'),
                'The User Verification Method is missing'
            );
        }

        $uvm = $outputs->get('uvm');
        //... Proceed with the output
    }
}
```
{% endcode %}

The easiest way to manage that is by using the creation and request profiles.

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    …
    creation_profiles:
        default:
            rp:
                name: 'My Application'
                id: 'example.com'
            extensions:
                uvm: true
    request_profiles:
        default:
            rp_id: 'example.com'
            extensions:
                uvm: true
```
{% endcode %}
