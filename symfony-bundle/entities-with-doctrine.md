---
description: Where the public keys and details are stored
---

# Credential Source Repository

The Credential Source can be stored the way you want. As the `Webauthn\PublicKeyCredentialSource` class can be converted into JSON, it could be stored in a filesystem.

In general, Symfony applications use Doctrine. That is why the bundle provides a way to use Doctrine as storage system.

### The Doctrine Entity

Hereafter an example of an entity.

This is the most simple example. Feel free to add custom fields that fits on your needs e.g. `created_at` _or `is_revoked`_.

{% code title="App/Entity/PublicKeyCredentialSource.php" %}
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
use Webauthn\PublicKeyCredentialSource as BasePublicKeyCredentialSource;
use Webauthn\TrustPath\TrustPath;

#[Table(name: "public_key_credential_sources")]
#[Entity(repositoryClass: PublicKeyCredentialSourceRepository::class)]
class PublicKeyCredentialSource extends BasePublicKeyCredentialSource
{
    #[Id]
    #[Column(type: "ulid", unique: true)]
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
We must override the method `saveCredentialSource` because we may receive `Webauthn\PublicKeyCredentialSource` objects instead of `App\Entity\PublicKeyCredentialSource`.
{% endhint %}

{% code title="App/Repository/PublicKeyCredentialSourceRepository.php" %}
```php
<?php

declare(strict_types=1);

namespace App\Repository;

use App\Entity\PublicKeyCredentialSource;
use Doctrine\Persistence\ManagerRegistry;
use Webauthn\Bundle\Repository\PublicKeyCredentialSourceRepository as BasePublicKeyCredentialSourceRepository;
use Webauthn\PublicKeyCredentialSource as BasePublicKeyCredentialSource;

final class PublicKeyCredentialSourceRepository extends BasePublicKeyCredentialSourceRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct($registry, PublicKeyCredentialSource::class);
    }

    public function saveCredentialSource(BasePublicKeyCredentialSource $publicKeyCredentialSource, bool $flush = true): void
    {
        if (!$publicKeyCredentialSource instanceof PublicKeyCredentialSource) {
            $publicKeyCredentialSource = new PublicKeyCredentialSource(
                $publicKeyCredentialSource->getPublicKeyCredentialId(),
                $publicKeyCredentialSource->getType(),
                $publicKeyCredentialSource->getTransports(),
                $publicKeyCredentialSource->getAttestationType(),
                $publicKeyCredentialSource->getTrustPath(),
                $publicKeyCredentialSource->getAaguid(),
                $publicKeyCredentialSource->getCredentialPublicKey(),
                $publicKeyCredentialSource->getUserHandle(),
                $publicKeyCredentialSource->getCounter()
            );
        }
        parent::saveCredentialSource($publicKeyCredentialSource, $flush);
    }
}
```
{% endcode %}

This repository should be declared as a Symfony service.

{% hint style="info" %}
With Symfony Flex, this is usually done automatically
{% endhint %}

{% code title="config/services.yaml" %}
```yaml
services:
    App\Repository\PublicKeyCredentialSourceRepository: ~
```
{% endcode %}
