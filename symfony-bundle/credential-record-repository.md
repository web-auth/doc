---
description: Where the public keys and details are stored
---

# Credential Record Repository

The Credential Record can be stored the way you want. As the `Webauthn\CredentialRecord` class can be converted into JSON, it could be stored in a filesystem.

It is up to you to create a credential record repository. This service shall implement `Webauthn\Bundle\Repository\CredentialRecordRepositoryInterface`.

{% hint style="warning" %}
**Renamed in v5.3.0:** `PublicKeyCredentialSource` has been renamed to `CredentialRecord` and `PublicKeyCredentialSourceRepositoryInterface` to `CredentialRecordRepositoryInterface`. The old names are deprecated and will be removed in 6.0.
{% endhint %}

{% hint style="info" %}
**Fixed in v5.3.7:** the bundle aliases the deprecated `PublicKeyCredentialSourceRepositoryInterface` to your repository only when that repository actually implements it. Symfony refuses an interface alias whose target does not implement the interface, so a repository following the deprecation and implementing `CredentialRecordRepositoryInterface` alone made the container invalid. Applications whose repository still implements the legacy interface keep the alias, and code type-hinting it keeps working.
{% endhint %}

{% hint style="warning" %}
Doctrine users: the field type for `transports` and `other_ui` changed from `array` to `json` (`array` is now deprecated) between the bundle v4.x and 5.0.

Please make sure to reflect the changes to your data model.

Here is an example for Postgres:

```sql
ALTER TABLE [/*TABLE NAME HERE*/] ALTER transports TYPE JSON USING transports::JSON
ALTER TABLE [/*TABLE NAME HERE*/] ALTER other_ui TYPE JSON USING other_ui::JSON

```
{% endhint %}

## Registration Capability

By default, the User Entity Repository is not able to register any user account. You can add this behavior by implementing the interface `Webauthn\Bundle\Repository\CanRegisterUserEntity`.

## Doctrine Repository

In general, Symfony applications use Doctrine, which is why the bundle provides a way to use Doctrine as storage system.

### The Doctrine Entity

Here is an example of an entity.

This is the most simple example. Feel free to add custom fields that fit your needs e.g. `created_at` or `is_revoked`.

{% code title="App/Entity/WebauthnCredential.php" lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

namespace App\Entity;

use App\Repository\WebauthnCredentialRepository;
use Doctrine\ORM\Mapping\Entity;
use Doctrine\ORM\Mapping\Column;
use Doctrine\ORM\Mapping\GeneratedValue;
use Doctrine\ORM\Mapping\Id;
use Doctrine\ORM\Mapping\Table;
use Symfony\Component\Uid\AbstractUid;
use Symfony\Component\Uid\Ulid;
use Webauthn\CredentialRecord;
use Webauthn\TrustPath\TrustPath;

#[Table(name: "webauthn_credentials")]
#[Entity(repositoryClass: WebauthnCredentialRepository::class)]
class WebauthnCredential extends CredentialRecord
{
    #[Id]
    #[Column(unique: true)]
    #[GeneratedValue(strategy: "NONE")]
    private string $id;

    public function __construct(
        string $publicKeyCredentialId,
        string $type,
        array $transports,
        string $attestationType,
        TrustPath $trustPath,
        AbstractUid $aaguid,
        string $credentialPublicKey,
        string $userHandle,
        int $counter
    )
    {
        $this->id = Ulid::generate();
        parent::__construct($publicKeyCredentialId, $type, $transports, $attestationType, $trustPath, $aaguid, $credentialPublicKey, $userHandle, $counter);
    }

    public function getId(): string
    {
        return $this->id;
    }
}
```
{% endcode %}

{% hint style="info" %}
Do not forget to update your database schema!
{% endhint %}

{% hint style="warning" %}
**New in v5.4.0:** the Doctrine mapping shipped by the bundle declares the new nullable `rpId` field of the Credential Record. Applications using that mapping have to generate and run a migration adding the column, which is nullable and empty for the existing rows.
{% endhint %}

## The Repository

{% hint style="warning" %}
**Deprecation Notice (v5.2.0):** The `DoctrineCredentialSourceRepository` class is deprecated and will be removed in version 6.0.0. You should create your own Doctrine-based repository implementation instead of extending the provided class. See the example below for guidance.
{% endhint %}

To ease the integration into your application, the bundle provides a concrete class that you can extend.

{% hint style="info" %}
In this following example, we extend that class and add a method to get all credentials for a specific user handle. Feel free to add your own methods.
{% endhint %}

{% hint style="warning" %}
We must override the method `saveCredentialRecord` because we may receive `Webauthn\CredentialRecord` objects instead of `App\Entity\WebauthnCredential`.
{% endhint %}

<pre class="language-php" data-title="App/Repository/WebauthnCredentialRepository.php" data-line-numbers><code class="lang-php">&#x3C;?php

declare(strict_types=1);

namespace App\Repository;

<strong>use App\Entity\WebauthnCredential;
</strong>use Doctrine\Persistence\ManagerRegistry;
use Webauthn\Bundle\Repository\DoctrineCredentialSourceRepository;
use Webauthn\CredentialRecord;

final class WebauthnCredentialRepository extends DoctrineCredentialSourceRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct($registry, WebauthnCredential::class);
    }

    public function saveCredentialRecord(CredentialRecord $credentialRecord): void
    {
        if (!$credentialRecord instanceof WebauthnCredential) {
            $credentialRecord = new WebauthnCredential(
                $credentialRecord->publicKeyCredentialId,
                $credentialRecord->type,
                $credentialRecord->transports,
                $credentialRecord->attestationType,
                $credentialRecord->trustPath,
                $credentialRecord->aaguid,
                $credentialRecord->credentialPublicKey,
                $credentialRecord->userHandle,
                $credentialRecord->counter
            );
        }
        parent::saveCredentialSource($credentialRecord);
    }
}

</code></pre>

This repository should be declared as a Symfony service.

{% hint style="info" %}
With Symfony autowiring and autoconfiguration, this is usually done automatically
{% endhint %}

{% code title="config/services.yaml" %}
```yaml
services:
    App\Repository\WebauthnCredentialRepository: ~
```
{% endcode %}
