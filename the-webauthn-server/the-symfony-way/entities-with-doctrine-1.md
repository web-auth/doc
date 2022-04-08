# User Entity Entity and Repository

## User Entity

Your application certainly has a user repository. Good news: there is no need to modify!

The User Entity Repository can be completely decoupled from the user entity used by your application.

This repository should be declared as a Symfony service and shall implement `Webauthn\Bundle\Repository\PublicKeyCredentialUserEntityRepository`.

Hereafter an example where the application User Repository is injected. This repository uses Doctrine and provides `findOneBy*` methods.

{% code title="src/Repository/PublicKeyCredentialUserEntityRepository.php" %}
```php
<?php

declare(strict_types=1);

namespace App\Repository;

use App\Entity\User;
use Symfony\Component\Uid\Ulid;
use Doctrine\ORM\EntityManagerInterface;
use Webauthn\PublicKeyCredentialUserEntity;
use Webauthn\Bundle\Repository\PublicKeyCredentialUserEntityRepository as PublicKeyCredentialUserEntityRepositoryInterface;

final class PublicKeyCredentialUserEntityRepository implements PublicKeyCredentialUserEntityRepositoryInterface
{
    /**
     * The UserRepository $userRepository is the Doctrine repository
     * that already exists in the application
     */
    public function __construct(private UserRepository $userRepository, private EntityManagerInterface $em)
    {
    }

    public function generateNextUserEntityId(): string
    {
        return Ulid::generate();
    }

    /**
     * This method saves the user or does nothing if the user already exists
     */
    public function saveUserEntity(PublicKeyCredentialUserEntity $userEntity): void
    {
        /** @var User|null $user */
        $user = $this->userRepository->findOneBy([
            'id' => $userEntity->getId(),
        ]);
        if ($user !== null) {
            return;
        }
        $user = new User($userEntity->getId(), $userEntity->getName(), $userEntity->getDisplayName());
        $this->em->persist($user);
        $this->em->flush();
    }

    public function findOneByUsername(string $username): ?PublicKeyCredentialUserEntity
    {
        /** @var User|null $user */
        $user = $this->userRepository->findOneBy([
            'username' => $username,
        ]);

        return $this->getUserEntity($user);
    }

    public function findOneByUserHandle(string $userHandle): ?PublicKeyCredentialUserEntity
    {
        /** @var User|null $user */
        $user = $this->userRepository->findOneBy([
            'id' => $userHandle,
        ]);

        return $this->getUserEntity($user);
    }

    /**
     * Converts a Symfony User (if any) into a Webauthn User Entity
     */
    private function getUserEntity(null|User $user): ?PublicKeyCredentialUserEntity
    {
        if ($user === null) {
            return null;
        }

        return new PublicKeyCredentialUserEntity(
            $user->getUsername(),
            $user->getUserIdentifier(),
            $user->getDisplayName(),
            null
        );
    }
}

```
{% endcode %}

{% hint style="info" %} Do not forget to add 2 methods on your User Entity {% endhint %}

{% code title="src/Entity/User.php" %}
```php
public function getUsername(): string
{
    return (string) $this->email;
}

public function getDisplayName(): string
{
    return (string) $this->email;
}
```
{% endcode %}
