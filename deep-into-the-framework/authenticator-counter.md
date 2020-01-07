# Authenticator Counter

The authenticators may have an internal counter. This feature is very helpful to detect cloned devices.

The default behaviour is to reject the assertions. This behaviour might cause some troubles as it could reject the real device whilst the fake one can continue to be used.

It is therefore required to go deeper in the protection of your application by logging the error and locking the associated account.

To do so , you have to create a custom Counter Checker and inject it to your Authenticator Assertion Response Validator. The checker must implement the interface `Webauthn\Counter\CounterChecker`.

```php
<?php

declare(strict_types=1);


namespace App\Service;

use App\SecuritySystem;
use Assert\Assertion;
use Throwable;
use Webauthn\PublicKeyCredentialSource;

final class CustomCounterChecker implements CounterChecker
{
    private $securitySystem;

    public function __construct(SecuritySystem $securitySystem)
    {
        $this->securitySystem = $securitySystem ;
    }

    public function check(PublicKeyCredentialSource $publicKeyCredentialSource, int $currentCounter): void
    {
        try {
            Assertion::greaterThan($currentCounter, $publicKeyCredentialSource->getCounter(), 'Invalid counter.');
        } catch (Throwable $throwable) {
            $this->securitySystem->fakeDeviceDetected($publicKeyCredentialSource);
            throw $throwable;
        }
    }
}
```

##  The Easy Way

```php
<?php

use Webauthn\Server;

$server = new Server(
    $rpEntity
    $publicKeyCredentialSourceRepository
);

// Set your handler here
$server->setCounterChecker(new CustomCounterChecker());
```

## The Hard Way

```php
$authenticatorAssertionResponseValidator = new AuthenticatorAssertionResponseValidator(
    $publicKeyCredentialSourceRepository,
    $tokenBindingHandler,
    $extensionOutputCheckerHandler,
    $coseAlgorithmManager,
    new CustomCounterChecker()
);
```

## The Symfony Way

{% code title="config/packages/webauthn.yaml" %}
```yaml
webauthn:
    counter_checker: App\Service\CustomCounterChecker
```
{% endcode %}

