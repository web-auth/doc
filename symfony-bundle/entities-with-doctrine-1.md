# User Entity Repository

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
use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Uid\Ulid;
use Webauthn\Bundle\Repository\PublicKeyCredentialUserEntityRepository as PublicKeyCredentialUserEntityRepositoryInterface;
use Webauthn\PublicKeyCredentialUserEntity;

final class PublicKeyCredentialUserEntityRepository implements PublicKeyCredentialUserEntityRepositoryInterface
{
    /**
     * The UserRepository $userRepository is the Doctrine repository
     * that already exists in the application
     */
    public function __construct(private UserRepository $userRepository)
    {
    }

    public function generateNextUserEntityId(): string {
        return Ulid::generate();
    }

    /**
      * This method saves the user or does nothing if the user already exists.
      * It may throw an exception. Just adapt it on your needs
      */
    public function saveUserEntity(PublicKeyCredentialUserEntity $userEntity): void
    {
        /** @var User|null $user */
        $user = $this->userRepository->findOneBy([
            'id' => $userEntity->getId(),
        ]);
        if (!$user instanceof UserInterface) {
            return;
        }
        $user = new User(
            $userEntity->getId(),
            $userEntity->getUserIdentifier(),
            $userEntity->getDisplayName() // Or any other similar method your entity may have
        );
        $this->userRepository->save($user); // Custom method to be added in your repository
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
