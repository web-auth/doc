# Authenticator Counter

The authenticators may have an internal counter. This feature is very helpful to detect cloned devices.

The default behaviour is to reject the assertions. This behaviour might cause some troubles as it could reject the real device whilst the fake one can continue to be used.

It is therefore required to go deeper in the protection of your application by logging the error and locking the associated account.

To do so , you have to create a custom Counter Checker and inject it to your Authenticator Assertion Response Validator. The checker must implement the interface `Webauthn\Counter\CounterChecker`.

{% code lineNumbers="true" %}
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
{% endcode %}

The Counter Checker service can be injected to your Ceremony Step Manager Factory.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\CeremonyStep\CeremonyStepManagerFactory;

$csmFactory = new CeremonyStepManagerFactory();
$csmFactory->setCounterChecker($customCounterChecker);
```
{% endcode %}
