# The Symfony Way

Symfony is a very popular framework and an official bundle is provided in the package `web-auth/webauthn-symfony-bundle`.

{% hint style="info" %}
If you use Laravel, you may be intersted in [this project: https://github.com/asbiin/laravel-webauthn](https://github.com/asbiin/laravel-webauthn)
{% endhint %}

If you are using Symfony Flex then the bundle will automatically be installed. Otherwise you need to add it in your `AppKernel.php` file:

{% code-tabs %}
{% code-tabs-item title="src/AppKernel.php" %}
```php
<?php

public function registerBundles()
{
    $bundles = [
        // ...
        new Webauthn\Bundle\WebauthnBundle(),
    ];
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Credential Source

#### The Doctrine Entity

With Doctrine, you have to indicate how to store the Credential Source objects. Hereafter an example an entity. In this example we add an entity `id`  and a custom field `created_at`. We also indicate the repository as we will have a custom one.

{% hint style="info" %}
As the ID must have a fixed length and because the `credentialId` field of `Webauthn\PublicKeyCredentialSource` haven’t such requirement and is a binary string, we need to declare our own `id` field.
{% endhint %}

{% code-tabs %}
{% code-tabs-item title="App/Entity/PublicKeyCredentialSource.php" %}
```php
<?php

declare(strict_types=1);

/*
 * The MIT License (MIT)
 *
 * Copyright (c) 2014-2019 Spomky-Labs
 *
 * This software may be modified and distributed under the terms
 * of the MIT license.  See the LICENSE file for details.
 */

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

    /**
     * @var \DateTimeImmutable
     * @ORM\Column(type="datetime_immutable")
     */
    private $createdAt;

    public function __construct(string $publicKeyCredentialId, string $type, array $transports, string $attestationType, TrustPath $trustPath, UuidInterface $aaguid, string $credentialPublicKey, string $userHandle, int $counter)
    {
        $this->id = Uuid::uuid4()->toString();
        $this->createdAt = new \DateTimeImmutable();
        parent::__construct($publicKeyCredentialId, $type, $transports, $attestationType, $trustPath, $aaguid, $credentialPublicKey, $userHandle, $counter);
    }

    public function getId(): string
    {
        return $this->id;
    }

    public function getCreatedAt(): \DateTimeImmutable
    {
        return $this->createdAt;
    }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

#### The Repository

To ease the integration into your application, the bundle provides a concrete class that you can extend.

{% hint style="info" %}
In this following example, we extend that class and add a method to get all credentials for a specific user handle. Feel free to add your own methods.
{% endhint %}

{% hint style="warning" %}
We must override the method saveCredentialSource because we may receive `Webauthn\PublicKeyCredentialSource` objects instead of `App\Entity\PublicKeyCredentialSource`.
{% endhint %}

{% code-tabs %}
{% code-tabs-item title="App/Repository/PublicKeyCredentialSourceRepository.php" %}
```php
<?php

declare(strict_types=1);

/*
 * The MIT License (MIT)
 *
 * Copyright (c) 2014-2019 Spomky-Labs
 *
 * This software may be modified and distributed under the terms
 * of the MIT license.  See the LICENSE file for details.
 */

namespace App\Repository;

use App\Entity\PublicKeyCredentialSource;
use App\Entity\User;
use Doctrine\Common\Persistence\ManagerRegistry;
use Webauthn\Bundle\Repository\PublicKeyCredentialSourceRepository as BasePublicKeyCredentialSourceRepository;
use Webauthn\PublicKeyCredentialSource as BasePublicKeyCredentialSource;

final class PublicKeyCredentialSourceRepository extends BasePublicKeyCredentialSourceRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct($registry, PublicKeyCredentialSource::class);
    }

    /**
     * @return PublicKeyCredentialSource[]
     */
    public function allForUser(User $user): array
    {
        $qb = $this->getEntityManager()->createQueryBuilder();

        return $qb->select('c')
            ->from($this->getClass(), 'c')
            ->where('c.userHandle = :user_handle')
            ->setParameter(':user_handle', $user->getUserHandle())
            ->getQuery()
            ->execute()
        ;
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
{% endcode-tabs-item %}
{% endcode-tabs %}

This repository should be declared as a Symfony service.

{% hint style="info" %}
With Symfony 4, this is usually done automatically
{% endhint %}

{% code-tabs %}
{% code-tabs-item title="config/services.yaml" %}
```yaml
services:
    App\Repository\PublicKeyCredentialSourceRepository: ~
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### User Entity

#### Doctrine Entity

In a Symfony application context, you usually have to manage several user entities. Thus, in the following example, the user entity class will extend the required calls and implements the interface provided by the Symfony Security component.

Feel free to add the necessary setters as well as other fields you need \(creation date, last update at…\).

**Please note that the ID of the user IS NOT generated by Doctrine and must be a string.** We highly recommend you to use UUIDs.

{% code-tabs %}
{% code-tabs-item title="app/Entity/User.php" %}
```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Bridge\Doctrine\Validator\Constraints\UniqueEntity;
use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Validator\Constraints as Assert;

/**
 * @ORM\Table(name="users")
 * @ORM\Entity(repositoryClass="App\Repository\UserRepository")
 * @UniqueEntity("name")
 */
class User extends PublicKeyCredentialUserEntity implements UserInterface
{
    /**
     * @ORM\Id
     * @ORM\Column(type="string", length=255)
     */
    protected $id;

    /**
     * @ORM\Column(type="string", length=255)
     * @Assert\Length(max = 100)
     */
    protected $name;

    /**
     * @ORM\Column(type="string", length=255)
     * @Assert\Length(max = 100)
     */
    protected $displayName;

    /**
     * @ORM\Column(type="array")
     */
    protected $roles;

    public function __construct(string $id, string $name, string $displayName, array $roles)
    {
        parent::__construct($name, $id, $displayName);
        $this->roles = $roles;
    }

    public function getRoles(): array
    {
        return array_unique($this->roles + ['ROLE_USER']);
    }

    public function getPassword(): void
    {
    }

    public function getSalt(): void
    {
    }

    public function getUsername(): ?string
    {
        return $this->name;
    }

    public function eraseCredentials(): void
    {
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

#### The Repository

The following example uses Doctrine to create, persist or perform queries using the User objects created above.

{% code-tabs %}
{% code-tabs-item title="app/Repository/UserRepository.php" %}
```php
<?php

declare(strict_types=1);

/*
 * The MIT License (MIT)
 *
 * Copyright (c) 2014-2019 Spomky-Labs
 *
 * This software may be modified and distributed under the terms
 * of the MIT license.  See the LICENSE file for details.
 */

namespace App\Repository;

use App\Entity\User;
use Doctrine\Common\Persistence\ManagerRegistry;
use Webauthn\Bundle\Repository\AbstractPublicKeyCredentialUserEntityRepository;
use Webauthn\PublicKeyCredentialUserEntity;

final class PublicKeyCredentialUserEntityRepository extends AbstractPublicKeyCredentialUserEntityRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct($registry, User::class);
    }

    public function createUserEntity(string $username, string $displayName, ?string $icon): PublicKeyCredentialUserEntity
    {
        return new User($username, $displayName, [], $icon);
    }

    public function saveUserEntity(PublicKeyCredentialUserEntity $userEntity): void
    {
        if (!$userEntity instanceof User) {
            $userEntity =  $this->createUserEntity(
                $userEntity->getName(),
                $userEntity->getDisplayName(),
                [],
                $userEntity->getIcon()
            );
        }

        parent::saveUserEntity($userEntity);
    }

    public function find(string $username): ?User
    {
        $qb = $this->getEntityManager()->createQueryBuilder();

        return $qb->select('u')
            ->from(User::class, 'u')
            ->where('u.name = :name')
            ->setParameter(':name', $username)
            ->setMaxResults(1)
            ->getQuery()
            ->getOneOrNullResult()
        ;
    }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

This repository should be declared as a Symfony service.

{% hint style="info" %}
With Symfony 4, this is usually done automatically
{% endhint %}

{% code-tabs %}
{% code-tabs-item title="config/services.yaml" %}
```yaml
services:
    App\Repository\UserRepository: ~
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Bundle Configuration



