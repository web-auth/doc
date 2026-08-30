# Cross Origin Authentication

Please [refer to this page](../../pure-php/advanced-behaviours/cross-origin-authentication.md) to know more about the Cross Origin Authentication.

## Configuration

The configuration of the allowed domains can be done as follows.

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    allowed_origins:
        - 'https://acme.com'
        - 'https://acme.fr'
        - 'android:apk-key-hash://your-app-hash'
        - 'ios:bundle-id://your.bundle.id'
    allow_subdomains: true
```
{% endcode %}

{% hint style="info" %}
**Changed in v5.3.3:** entries that are not URLs, such as `android:apk-key-hash:<base64>`, are compared verbatim with the origin sent by the client. Entries made of a host only are normalized to `https://`.
{% endhint %}

## Top Origin Validation (Cross-Origin iframes)

When the ceremony runs in a cross-origin iframe, the client data contains the origin of the embedding page in a `topOrigin` field.

{% hint style="danger" %}
**Changed in v5.3.4:** such a response is rejected unless a top origin validator is configured. Applications that do not use iframes are not affected, as a same-origin ceremony carries no `topOrigin`.
{% endhint %}

Write a service implementing `Webauthn\CeremonyStep\TopOriginValidator`:

{% code title="src/Security/MyTopOriginValidator.php" lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

namespace App\Security;

use Webauthn\CeremonyStep\TopOriginValidator;
use Webauthn\Exception\AuthenticatorResponseVerificationException;

final readonly class MyTopOriginValidator implements TopOriginValidator
{
    public function validate(string $topOrigin): void
    {
        $allowed = [
            'https://app.example.com',
            'https://dashboard.example.com',
        ];
        if (!in_array($topOrigin, $allowed, true)) {
            throw AuthenticatorResponseVerificationException::create(
                'The top origin is not allowed.'
            );
        }
    }
}
```
{% endcode %}

Then declare it with the `top_origin_validator` option:

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    top_origin_validator: 'App\Security\MyTopOriginValidator'
```
{% endcode %}

{% hint style="info" %}
The option is an alias to your service. A service registered under the `Webauthn\CeremonyStep\TopOriginValidator` identifier, which autowiring does by default when a single implementation exists, is picked up without any configuration. The validator applies to both the creation and the request ceremonies.
{% endhint %}

See [the pure PHP documentation](../../pure-php/advanced-behaviours/cross-origin-authentication.md#top-origin-validation-cross-origin-iframes) for the built-in `HostTopOriginValidator` and more details.

## Allowed Origins Endpoint

When the allowed\_origins parameter is set, the path `/.well-known/webauthn` is enabled. This path returns a `JSON` object with allowed domains.
