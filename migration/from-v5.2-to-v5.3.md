---
description: Upgrade guide from 5.2 to 5.3
---

# From 5.2 to 5.3

{% hint style="success" %}
No backward compatibility break. The deprecated classes, methods and nodes keep working until 6.0.0.
{% endhint %}

Version 5.3 aligns the vocabulary of the library with the WebAuthn Level 3 specification: `PublicKeyCredentialSource` becomes `CredentialRecord`. It also removes the Relying Party name from the required data and moves the Stimulus controllers to npm.

## PublicKeyCredentialSource becomes CredentialRecord

{% hint style="warning" %}
**Deprecated in v5.3.0**
{% endhint %}

`Webauthn\PublicKeyCredentialSource` is renamed `Webauthn\CredentialRecord`, which is the name used by the specification for the data a Relying Party stores about a credential. The old class extends the new one, so both types can coexist during the migration, and every framework component accepts both.

```php
# Before (deprecated)
use Webauthn\PublicKeyCredentialSource;

$credential = new PublicKeyCredentialSource(/* ... */);

# After
use Webauthn\CredentialRecord;

$credential = new CredentialRecord(/* ... */);
```

The renaming applies to the whole surface:

| Deprecated since 5.3.0 | Replacement |
|---|---|
| `Webauthn\PublicKeyCredentialSource` | `Webauthn\CredentialRecord` |
| `Webauthn\Bundle\Repository\PublicKeyCredentialSourceRepositoryInterface` | `Webauthn\Bundle\Repository\CredentialRecordRepositoryInterface` |
| `Webauthn\Bundle\Repository\CanSaveCredentialSource` | `Webauthn\Bundle\Repository\CanSaveCredentialRecord` |
| `saveCredentialSource()` | `saveCredentialRecord()` |
| The `credentialSource` property and `getPublicKeyCredentialSource()` method of the validation and backup events | The `credentialRecord` property |

### Repository migration

Update your repository to the new interface. The methods are the same, so there is nothing else to change:

```php
// Before (deprecated)
use Webauthn\Bundle\Repository\PublicKeyCredentialSourceRepositoryInterface;

class MyCredentialRepository implements PublicKeyCredentialSourceRepositoryInterface
{
    // ...
}

// After
use Webauthn\Bundle\Repository\CredentialRecordRepositoryInterface;

class MyCredentialRepository implements CredentialRecordRepositoryInterface
{
    // No other change needed, the methods are identical
}
```

A repository can implement both interfaces during a gradual migration, for instance when your own code still type-hints the legacy one:

```php
use Webauthn\Bundle\Repository\CredentialRecordRepositoryInterface;
use Webauthn\Bundle\Repository\PublicKeyCredentialSourceRepositoryInterface;

class MyCredentialRepository implements
    CredentialRecordRepositoryInterface,
    PublicKeyCredentialSourceRepositoryInterface
{
    // Both interfaces declare the same methods, no duplication needed
}
```

{% hint style="info" %}
**Fixed in v5.3.7:** the bundle aliases the deprecated `PublicKeyCredentialSourceRepositoryInterface` to your repository only when that repository actually implements it. Symfony refuses an interface alias whose target does not implement the interface, so a repository following the deprecation and implementing `CredentialRecordRepositoryInterface` alone made the container invalid. Applications whose repository still implements the legacy interface keep the alias, and code type-hinting it keeps working.
{% endhint %}

### Entity and database

The database schema does not change. Both classes share the very same Doctrine mapping:

* `publicKeyCredentialId` (base64, unique, length 250)
* `type` (string)
* `transports` (json)
* `attestationType` (string)
* `trustPath` (trust\_path type)
* `aaguid` (aaguid type, length 36)
* `credentialPublicKey` (base64)
* `userHandle` (string)
* `counter` (integer)
* `otherUI` (json, nullable)
* `backupEligible` (boolean, nullable)
* `backupStatus` (boolean, nullable)
* `uvInitialized` (boolean, nullable)

**No database migration is required.** Change the parent class of your entity and you are done:

```php
// Before
class MyCredential extends PublicKeyCredentialSource { }

// After
class MyCredential extends CredentialRecord { }
```

{% hint style="info" %}
Version 5.4 adds an optional `rpId` member to the credential record, which does require a schema update. See [From 5.3 to 5.4](from-v5.3-to-v5.4.md).
{% endhint %}

### Converting existing objects

When your code has to hand a `PublicKeyCredentialSource` to a component that is not migrated yet, or the other way around, use the `CredentialRecordConverter` utility:

```php
use Webauthn\Util\CredentialRecordConverter;

$credentialRecord = CredentialRecordConverter::toCredentialRecord($publicKeyCredentialSource);
$publicKeyCredentialSource = CredentialRecordConverter::toPublicKeyCredentialSource($credentialRecord);

$credentialRecords = CredentialRecordConverter::toCredentialRecords($publicKeyCredentialSources);
$publicKeyCredentialSources = CredentialRecordConverter::toPublicKeyCredentialSources($credentialRecords);
```

### Migration checklist

* [ ] Update your repository to `CredentialRecordRepositoryInterface` and `CanSaveCredentialRecord`
* [ ] Rename `saveCredentialSource()` to `saveCredentialRecord()`
* [ ] Update your entity to extend `CredentialRecord`
* [ ] Update the type hints and the event listeners reading `credentialSource`
* [ ] Remove every remaining reference to the deprecated names before upgrading to 6.0.0

## Other deprecations

### PublicKeyCredentialRpEntity.name

{% hint style="warning" %}
**Deprecated in v5.3.0**
{% endhint %}

The `name` property of `PublicKeyCredentialRpEntity` is deprecated and will be removed in version 6.0.0. According to the WebAuthn Level 3 specification, the Relying Party name is no longer required.

```php
# Before (deprecated)
$rpEntity = PublicKeyCredentialRpEntity::create('My Application', 'example.com');

# After
$rpEntity = PublicKeyCredentialRpEntity::create('', 'example.com');
```

{% hint style="info" %}
**Changed in v5.3.7:** the deprecation is carried by `PublicKeyCredentialRpEntity` only. `PublicKeyCredentialUserEntity.name` is **not** deprecated: the property moves from the abstract parent class down to the user entity in 6.0.0 and keeps working exactly as before. Until 5.3.6 the tag sat on the parent class, so static analysis reported every read of a user entity name as deprecated.
{% endhint %}

The `name` member is required by the W3C IDL. Since v5.3.7 the serializer writes the Relying Party ID in that member when the entity has an empty name, so the payload sent to the browser stays valid without any action on your side.

With the Symfony bundle, drop the `rp.name` node from your creation profiles:

```yaml
webauthn:
    creation_profiles:
        default:
            rp:
                id: '%env(RELYING_PARTY_ID)%'
```

Between 5.3.3 and 5.3.6 the bundle copied `rp.id` into `rp.name` on the entity itself, which triggered the deprecation on every options request, including for the applications that had already removed the node. Since 5.3.7 the fallback is applied at serialization time and the deprecation is only triggered by an application that really sets a name.

### createFormJson

{% hint style="warning" %}
**Deprecated in v5.3.0**
{% endhint %}

`CollectedClientData::createFormJson()` is deprecated and will be removed in 6.0.0. Use the standard Symfony Serializer to deserialize the credential responses instead. See [Input Loading](../pure-php/input-loading.md).

### Authenticator Transport CABLE

{% hint style="warning" %}
**Deprecated in v5.3.0**
{% endhint %}

The constant `AUTHENTICATOR_TRANSPORT_CABLE` is deprecated and will be removed in version 6.0.0. Use `AUTHENTICATOR_TRANSPORT_HYBRID`, the spec-aligned successor for caBLE (cloud-assisted BLE).

```php
# Before (deprecated)
use Webauthn\PublicKeyCredentialDescriptor;

$transport = PublicKeyCredentialDescriptor::AUTHENTICATOR_TRANSPORT_CABLE;

# After
use Webauthn\PublicKeyCredentialDescriptor;

$transport = PublicKeyCredentialDescriptor::AUTHENTICATOR_TRANSPORT_HYBRID;
```

### Composer package `web-auth/webauthn-stimulus`

{% hint style="warning" %}
**Deprecated in v5.3.0, removed in v6.0.0**
{% endhint %}

The dedicated PHP package `web-auth/webauthn-stimulus` (the Symfony Flex/AssetMapper wrapper around the Stimulus controllers) is deprecated. The same JavaScript is now published to npm as [`@web-auth/webauthn-stimulus`](https://www.npmjs.com/package/@web-auth/webauthn-stimulus) and that is the only package that will keep being maintained in 6.0.0.

Migrate your application before upgrading to 6.0.0:

```bash
# 1. Drop the Composer wrapper
composer remove web-auth/webauthn-stimulus

# 2. Pin the npm package, pick one
php bin/console importmap:require @web-auth/webauthn-stimulus   # AssetMapper
npm install @web-auth/webauthn-stimulus                          # Encore / Vite / esbuild
```

Then register the controllers from your Stimulus bootstrap file (`assets/bootstrap.js` with the default AssetMapper recipe) under their package-prefixed identifiers:

```javascript
import { Application } from '@hotwired/stimulus';
import {
    AuthenticationController,
    RegistrationController,
    WebauthnController,
} from '@web-auth/webauthn-stimulus';

const app = Application.start();
app.register('web-auth--webauthn-stimulus--authentication', AuthenticationController);
app.register('web-auth--webauthn-stimulus--registration', RegistrationController);
app.register('web-auth--webauthn-stimulus', WebauthnController);
```

{% hint style="danger" %}
**Do not add `@web-auth/webauthn-stimulus` to `assets/controllers.json`.** Symfony UX `StimulusBundle` resolves `controllers.json` entries against installed Composer packages, so it throws `Could not find package "web-auth/webauthn-stimulus" referred to from controllers.json` once the Composer wrapper is gone. Register from JavaScript instead, as shown above.
{% endhint %}

Your Twig templates do not need any change: `stimulus_controller('@web-auth/webauthn-stimulus/authentication')` still resolves to the `web-auth--webauthn-stimulus--authentication` identifier you just registered.

### SimpleFakeCredentialGenerator without a secret

{% hint style="warning" %}
**Deprecated in v5.3.5**
{% endhint %}

Instantiating `Webauthn\SimpleFakeCredentialGenerator` without a secret is deprecated and a non-empty secret will be required in 6.0.0. Without a secret, the generated fake credentials only depend on the username, so they are identical across applications and an attacker can tell a fake credential from a real one.

```php
# Before (deprecated)
$generator = new SimpleFakeCredentialGenerator();

# After
$generator = new SimpleFakeCredentialGenerator($secret);
```

See [Fake Credentials](../symfony-bundle/advanced-behaviors/fake-credentials.md).

## Changed behaviors

### Options Handlers signature

{% hint style="warning" %}
**Changed in v5.3.0**
{% endhint %}

The `CreationOptionsHandler` and `RequestOptionsHandler` interfaces now accept an optional `?Request $request` parameter. If you implement these interfaces, you must update the signature of your methods.

```php
# Before
use Symfony\Component\HttpFoundation\Response;
use Webauthn\PublicKeyCredentialCreationOptions;
use Webauthn\PublicKeyCredentialUserEntity;

class MyCreationOptionsHandler implements CreationOptionsHandler
{
    public function onCreationOptions(
        PublicKeyCredentialCreationOptions $options,
        PublicKeyCredentialUserEntity $userEntity,
    ): Response {
        // ...
    }
}

# After
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Webauthn\PublicKeyCredentialCreationOptions;
use Webauthn\PublicKeyCredentialUserEntity;

class MyCreationOptionsHandler implements CreationOptionsHandler
{
    public function onCreationOptions(
        PublicKeyCredentialCreationOptions $options,
        PublicKeyCredentialUserEntity $userEntity,
        ?Request $request = null,
    ): Response {
        // ...
    }
}
```

The same applies to `RequestOptionsHandler::onRequestOptions()`.

## Other changes

### New Authenticator Transports

{% hint style="info" %}
**Added in v5.3.0**
{% endhint %}

`PublicKeyCredentialDescriptor` exposes two new transport constants in addition to the historic `usb`, `nfc`, `ble` and `internal`:

* `AUTHENTICATOR_TRANSPORT_SMART_CARD` (`smart-card`)
* `AUTHENTICATOR_TRANSPORT_HYBRID` (`hybrid`, replaces `cable`)

All seven values are referenced by `PublicKeyCredentialDescriptor::AUTHENTICATOR_TRANSPORTS`.
