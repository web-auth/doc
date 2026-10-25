# Cross origin authentication

Please [refer to this page](../../pure-php/advanced-behaviours/dealing-with-localhost.md) to know more about the Cross Origin Authentication.

## Configuration

The configuration of the allowed domains can be done as follows.

```yaml title="config/packages/webauthn.yaml" linenums="1"
webauthn:
    allowed_origins:
        - 'https://acme.com'
        - 'https://acme.fr'
        - 'android:apk-key-hash://your-app-hash'
        - 'ios:bundle-id://your.bundle.id'
    allow_subdomains: true
```

## Top origin validation (cross-origin iframes)

If your authentication page is embedded in a cross-origin iframe, you can enable top origin validation by registering a service that implements `Webauthn\CeremonyStep\TopOriginValidator`.

When a `TopOriginValidator` service is available in the container, it is automatically used for both creation and request ceremonies.

```php title="src/Security/MyTopOriginValidator.php" linenums="1"
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

!!! info

    If no `TopOriginValidator` service is registered, the top origin is not validated. See [the pure PHP documentation](../../pure-php/advanced-behaviours/dealing-with-localhost.md#top-origin-validation-cross-origin-iframes) for more details.

## Allowed origins endpoint

When the allowed\_origins parameter is set, the path `/.well-known/webauthn` is enabled. This path returns a `JSON` object with allowed domains.
