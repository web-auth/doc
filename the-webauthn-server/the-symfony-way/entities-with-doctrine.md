# Credential Source Entity and Repository

## Credential Source

### The Doctrine Entity

With Doctrine, you have to indicate how to store the Credential Source objects. Hereafter an example of an entity. In this example we add an entity `id` and a custom field `created_at`. We also indicate the repository as we will have a custom one.

{% hint style="info" %}
As the ID must have a fixed length and because the `credentialId` field of `Webauthn\PublicKeyCredentialSource` hasn’t such a requirement and is a binary string, thus we need to declare our own `id` field.
{% endhint %}

{% code title="App/Entity/PublicKeyCredentialSource.php" %}
```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Ramsey\Uuid\Uuid;
use Ramsey\Uuid\UuidInterface;
use Webauthn\PublicKeyCredentialSource as BasePublicKeyCredentialSource;
use Webauthn\TrustPath\TrustPath;

/**
 * @ORM\Table(name="public_key_credential_sources")
 * @ORM\Entity(repositoryClass="App\Repository\PublicKeyCredentialSourceRepository")
 */
class PublicKeyCredentialSource extends BasePublicKeyCredentialSource
{
    /**
     * @var string
     * @ORM\Id
     * @ORM\Column(type="string", length=100)
     * @ORM\GeneratedValue(strategy="NONE")
     */
    private $id;

    public function __construct(string $publicKeyCredentialId, string $type, array $transports, string $attestationType, TrustPath $trustPath, UuidInterface $aaguid, string $credentialPublicKey, string $userHandle, int $counter)
    {
        $this->id = Uuid::uuid4()->toString();
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

### The Repository

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
use Doctrine\Common\Persistence\ManagerRegistry;
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
