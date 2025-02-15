---
description: Where the public keys and details are stored
---

# Credential Source Repository

The Credential Source can be stored the way you want. As the `Webauthn\PublicKeyCredentialSource` class can be converted into JSON, it could be stored in a filesystem.

It is up to you to create a credential source repository. This service shall implement `Webauthn\Bundle\Repository\PublicKeyCredentialSourceRepositoryInterface`.

{% hint style="warning" %}
Doctrine users: the field type for `transports` and `other_ui` changed from `array` to `json` (`array` is now deprecated) between the bundle v4.x and 5.0.

Please make sure to reflect the changes to your data model.

Hereafter an example for Postgres:

```sql
ALTER TABLE [/*TABLE NAME HERE*/] ALTER transports TYPE JSON USING transports::JSON
ALTER TABLE [/*TABLE NAME HERE*/] ALTER other_ui TYPE JSON USING other_ui::JSON

```
{% endhint %}

## Registration Capability

By default, the User Entity Repository is not able to register any user account. You can add this behaviour by implementing the interface `Webauthn\Bundle\Repository\CanRegisterUserEntity`.

## Doctrine Repository

In general, Symfony applications use Doctrine and is why the bundle provides a way to use Doctrine as storage system.

### The Doctrine Entity

Hereafter an example of an entity.

This is the most simple example. Feel free to add custom fields that fits on your needs e.g. `created_at` or `is_revoked`.

{% code title="App/Entity/WebauthnCredential.php" lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

namespace App\Entity;

use App\Repository\PublicKeyCredentialSourceRepository;
use Doctrine\ORM\Mapping\Entity;
use Doctrine\ORM\Mapping\Column;
use Doctrine\ORM\Mapping\GeneratedValue;
use Doctrine\ORM\Mapping\Id;
use Doctrine\ORM\Mapping\Table;
use Symfony\Component\Uid\AbstractUid;
use Symfony\Component\Uid\Ulid;
use Webauthn\PublicKeyCredentialSource;
use Webauthn\TrustPath\TrustPath;

#[Table(name: "webauthn_credentials")]
#[Entity(repositoryClass: WebauthnCredentialRepository::class)]
class WebauthnCredential extends PublicKeyCredentialSource
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

## The Repository

To ease the integration into your application, the bundle provides a concrete class that you can extend.

{% hint style="info" %}
In this following example, we extend that class and add a method to get all credentials for a specific user handle. Feel free to add your own methods.
{% endhint %}

{% hint style="warning" %}
We must override the method `saveCredentialSource` because we may receive `Webauthn\PublicKeyCredentialSource` objects instead of `App\Entity\WebauthnCredential`.
{% endhint %}

<pre class="language-php" data-title="App/Repository/WebauthnCredentialRepository.php" data-line-numbers><code class="lang-php">&#x3C;?php

declare(strict_types=1);

namespace App\Repository;

<strong>use App\Entity\WebauthnCredential;
</strong>use Doctrine\Persistence\ManagerRegistry;
use Webauthn\Bundle\Repository\DoctrineCredentialSourceRepository;
use Webauthn\PublicKeyCredentialSource;

final class WebauthnCredentialRepository extends DoctrineCredentialSourceRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct($registry, WebauthnCredential::class);
    }

    public function saveCredentialSource(PublicKeyCredentialSource $publicKeyCredentialSource): void
    {
        if (!$publicKeyCredentialSource instanceof WebauthnCredential) {
            $publicKeyCredentialSource = new WebauthnCredential(
                $publicKeyCredentialSource->publicKeyCredentialId,
                $publicKeyCredentialSource->type,
                $publicKeyCredentialSource->transports,
                $publicKeyCredentialSource->attestationType,
                $publicKeyCredentialSource->trustPath,
                $publicKeyCredentialSource->aaguid,
                $publicKeyCredentialSource->credentialPublicKey,
                $publicKeyCredentialSource->userHandle,
                $publicKeyCredentialSource->counter
            );
        }
        parent::saveCredentialSource($publicKeyCredentialSource);
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
