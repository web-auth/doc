# Backup Events

{% hint style="info" %}
**New in v5.3.0**
{% endhint %}

The framework dispatches PSR-14 events when the backup eligibility or backup status flags of a credential change during authentication. These events allow you to react to changes in credential backup state, which is important for security monitoring and user guidance.

## Background

WebAuthn authenticators report two backup-related flags:

* **BE (Backup Eligible)**: Indicates whether the authenticator is capable of backing up the credential (e.g., synced passkeys via iCloud Keychain, Google Password Manager)
* **BS (Backup Status)**: Indicates whether the credential is currently backed up

Changes in these flags can signal important security events:

* A credential becoming backup-eligible means the user may have synced their passkey
* A credential losing its backup status could mean the user should register additional authenticators for redundancy

## Events

### BackupEligibilityChangedEvent

Dispatched when the backup eligibility (BE) flag changes between authentications.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\Event\BackupEligibilityChangedEvent;

class BackupEligibilityListener
{
    public function __invoke(BackupEligibilityChangedEvent $event): void
    {
        $credentialRecord = $event->credentialRecord;
        $previousValue = $event->previousValue; // ?bool
        $newValue = $event->newValue;           // ?bool

        if ($newValue === true && $previousValue !== true) {
            // Credential has become backup-eligible
            // Log this change for auditing purposes
        }
    }
}
```
{% endcode %}

### BackupStatusChangedEvent

Dispatched when the backup status (BS) flag changes between authentications.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\Event\BackupStatusChangedEvent;

class BackupStatusListener
{
    public function __invoke(BackupStatusChangedEvent $event): void
    {
        $credentialRecord = $event->credentialRecord;
        $previousValue = $event->previousValue; // ?bool
        $newValue = $event->newValue;           // ?bool

        if ($previousValue === true && $newValue === false) {
            // Credential is no longer backed up
            // Consider prompting the user to register an additional authenticator
        }
    }
}
```
{% endcode %}

## Registering Event Listeners

### Pure PHP (PSR-14)

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\Event\BackupEligibilityChangedEvent;
use Webauthn\Event\BackupStatusChangedEvent;

// Using any PSR-14 compatible event dispatcher
$eventDispatcher->addListener(
    BackupEligibilityChangedEvent::class,
    new BackupEligibilityListener()
);

$eventDispatcher->addListener(
    BackupStatusChangedEvent::class,
    new BackupStatusListener()
);
```
{% endcode %}

### Symfony

With the Symfony bundle, register your listeners as services. With autoconfiguration, they are registered automatically.

{% code title="config/services.yaml" lineNumbers="true" %}
```yaml
services:
    App\EventListener\BackupStatusListener:
        tags:
            - { name: kernel.event_listener, event: Webauthn\Event\BackupStatusChangedEvent }
    App\EventListener\BackupEligibilityListener:
        tags:
            - { name: kernel.event_listener, event: Webauthn\Event\BackupEligibilityChangedEvent }
```
{% endcode %}

## Security Considerations

* **Monitor backup status loss**: If a credential transitions from backed up to not backed up, the user may have lost their backup. Consider prompting them to register additional authenticators.
* **Audit backup eligibility changes**: Unexpected changes in backup eligibility may indicate authenticator changes worth logging.
* **Do not block authentication**: These events are informational. Do not reject authentication attempts based on backup flag changes.

## See Also

* [Credential Record](../../prerequisites/credential-record.md) - Credential properties including backup flags
* [Authenticator Counter](authenticator-counter.md) - Another security monitoring mechanism
