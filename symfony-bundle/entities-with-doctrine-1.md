# User Entity Repository

## User Entity

Your application certainly has a user repository. Good news: there is no need to modify it!

The User Entity Repository can be completely decoupled from the user entity used by your application.

This repository should be declared as a Symfony service and shall implement `Webauthn\Bundle\Repository\PublicKeyCredentialUserEntityRepositoryInterface`.

Hereafter an example where the application User Repository is injected. This repository uses Doctrine and provides `findOneBy*` methods.

{% code title="src/Repository/WebauthnUserEntityRepository.php" %}
```php
<?php

declare(strict_types=1);

namespace App\Repository;

use App\Entity\User;
use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Uid\Ulid;
use Webauthn\Bundle\Repository\PublicKeyCredentialUserEntityRepositoryInterface;
use Webauthn\PublicKeyCredentialUserEntity;

final class WebauthnUserEntityRepository implements PublicKeyCredentialUserEntityRepositoryInterface
{
    /**
     * The UserRepository $userRepository is the repository
     * that already exists in the application
     */
    public function __construct(private UserRepository $userRepository)
    {
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

## Registration Capability

By default, the User Entity Repository is not able to register any user account. You can add this behaviour by implementing the interface `Webauthn\Bundle\Repository\CanRegisterUserEntity`.
