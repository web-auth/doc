# Authenticator Counter

The authenticators may have an internal counter. This feature is very helpful to detect cloned devices.

The default behavior is to reject the assertions. This might cause some troubles as it could reject the real device whilst the fake one can continue to be used. You may also want to log the error, warn administrators or lock the associated user account.

To do so, you have to create a custom Counter Checker and inject it into your Authenticator Assertion Response Validator. The checker must implement the interface `Webauthn\Counter\CounterChecker`.

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    counter_checker: App\Service\CustomCounterChecker
```
{% endcode %}

The following example is fictitious and shows how to lock a user, log the error and throw an exception.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

namespace Acme\Service;

use Assert\Assertion;
use Psr\Log\LoggerInterface;
use Psr\Log\NullLogger;
use Throwable;
use Webauthn\CredentialRecord;

final class CustomCounterChecker implements CounterChecker
{
    public function __construct(private UserRepository $userRepository)
    {
    }

    public function check(CredentialRecord $credentialRecord, int $currentCounter): void
    {
        if ($currentCounter > $credentialRecord->counter) {
            return;
        }

        $userId = $credentialRecord->userHandle;
        $this->userRepository->lockUserWithId($userId);
        throw new CustomSecurityException('Invalid counter. User is now locked.');
    }
}
```
{% endcode %}
