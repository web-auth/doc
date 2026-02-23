# Authenticator Counter

The authenticators may have an internal counter. This feature is very helpful to detect cloned devices.

The default behavior is to reject the assertions. This might cause some troubles as it could reject the real device whilst the fake one can continue to be used.

It is therefore required to go deeper in the protection of your application by logging the error and locking the associated account.

To do so, you have to create a custom Counter Checker and inject it into your Authenticator Assertion Response Validator. The checker must implement the interface `Webauthn\Counter\CounterChecker`.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);


namespace App\Service;

use App\SecuritySystem;
use Throwable;
use Webauthn\Counter\CounterChecker;
use Webauthn\CredentialRecord;

final class CustomCounterChecker implements CounterChecker
{
    public function __construct(private SecuritySystem $securitySystem)
    {
    }

    public function check(CredentialRecord $credentialRecord, int $currentCounter): void
    {
        try {
            assert($currentCounter > $credentialRecord->counter, 'Invalid counter.');
        } catch (Throwable $throwable) {
            $this->securitySystem->fakeDeviceDetected($credentialRecord);
            throw $throwable;
        }
    }
}
```
{% endcode %}

The Counter Checker service can be injected into your Ceremony Step Manager Factory.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\CeremonyStep\CeremonyStepManagerFactory;

$csmFactory = new CeremonyStepManagerFactory();
$csmFactory->setCounterChecker($customCounterChecker);
```
{% endcode %}
