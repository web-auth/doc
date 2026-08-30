# Backup Events

{% hint style="info" %}
**New in v5.3.0**
{% endhint %}

The framework dispatches PSR-14 events when the persisted indicators of a credential record change during authentication: the backup eligibility and backup status flags, and, since v5.4.0, the user verification indicator. These events allow you to react to those changes, which matters for security monitoring and user guidance.

## Background

WebAuthn authenticators report two backup-related flags:

* **BE (Backup Eligible)**: Indicates whether the authenticator is capable of backing up the credential (e.g., synced passkeys via iCloud Keychain, Google Password Manager)
* **BS (Backup Status)**: Indicates whether the credential is currently backed up

{% hint style="info" %}
**Fixed in v5.3.6:** `AuthenticatorData::getReservedForFutureUse2()` reported the bits 3 and 4 as reserved, although WebAuthn Level 3 assigns them to BE and BS. A synced passkey with both flags set returned `24` instead of `0`. The mask is now restricted to the only bit that is still reserved. Use `isBackupEligible()` and `isBackedUp()` to read the backup flags. The reserved-for-future-use accessors themselves are deprecated since v5.4.0, see [the migration guide](../../migration/from-v5.3-to-v5.4.md).
{% endhint %}

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

### UvInitializedChangedEvent

{% hint style="info" %}
**New in v5.4.0**
{% endhint %}

Dispatched when the persisted `uvInitialized` indicator of a credential record changes during an assertion ceremony.

The specification states that updating `uvInitialized` from `false` to `true` should require authorization by an additional authentication factor equivalent to WebAuthn user verification. The validator performs the transition as soon as an assertion arrives with the UV flag set, and this event is the hook where that additional verification belongs. When it cannot be obtained, revert the value on the credential record before it is persisted.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\Event\UvInitializedChangedEvent;

class UvInitializedListener
{
    public function __invoke(UvInitializedChangedEvent $event): void
    {
        $credentialRecord = $event->credentialRecord;
        $previousValue = $event->previousValue; // ?bool
        $newValue = $event->newValue;           // ?bool

        if ($previousValue !== true && $newValue === true) {
            // The credential now claims user verification.
            // Apply your step-up verification here, and revert the value
            // on the credential record if it cannot be obtained:
            // $credentialRecord->uvInitialized = $previousValue;
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
