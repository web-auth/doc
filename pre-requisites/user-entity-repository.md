# User Entity And Repository

## User Entity

A User Entity object represents a user in the Webauthn context. It has the following constraints:

* The user ID must be unique and must be a string,
* The username must be unique,

Hereafter a minimalist example of user entity:

```php
<?php

use Webauthn\PublicKeyCredentialUserEntity;

$userEntity = new PublicKeyCredentialUserEntity(
    'john.doe',                             // Username
    'ea4e7b55-d8d0-4c7e-bbfa-78ca96ec574c', // ID
    'John Doe'                              // Display name
);
```

{% hint style="info" %}
The username can be composed of any displayable characters, including emojies. Username "😝**🥰**😔" is perfectly valid.
{% endhint %}

{% hint style="warning" %}
For privacy reasons, it is not recommended to use the e-mail as username.
{% endhint %}

As for the `rp` Entity, the User Entity may have an icon. This icon must also be secured.

```php
<?php

use Webauthn\PublicKeyCredentialUserEntity;

$userEntity = new PublicKeyCredentialUserEntity(
    'john.doe',
    'ea4e7b55-d8d0-4c7e-bbfa-78ca96ec574c',
    'John Doe',
    'data:image/png;base64,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'
);
```

{% hint style="info" %}
The Webauthn specification does not set any limit for the length of the icon.
{% endhint %}

{% hint style="warning" %}
The icon may be ignored by browsers, especially if its length is greater than 128 bytes.
{% endhint %}

### Repository

The User Entity Repository manages all Webauthn users of your application.

There is no interface to implement or abstract class to extend so that it should be easy to integrate it in your application. You may already have a user repository.

{% hint style="success" %}
Whatever the database you use(MySQL, pgSQL…), it is not necessary to create relationships between your users and the Credential Sources.
{% endhint %}

Hereafter an example of a User Entity repository. In this example we suppose you already have methods to find users using their username or ID.

{% code title="Acme\Repository\PublicKeyCredentialUserEntityRepository.php" %}
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

namespace Acme\Repository;

use Webauthn\PublicKeyCredentialUserEntity;

final class PublicKeyCredentialUserEntityRepository
{
    public function findWebauthnUserByUsername(string $username): ?PublicKeyCredentialUserEntity
    {
        //We suppose you already have a method to find a user using its username
        $user = $this->findOneBy(['username' => $username]);
        if (null === $user) {
            return null;
        }

        return $this->createUserEntity($user);
    }

    public function findWebauthnUserByUserHandle(string $userHandle): ?PublicKeyCredentialUserEntity
    {
        //We suppose you already have a method to find a user using its ID
        $user = $this->findOneBy(['id' => $userHandle]);
        if (null === $user) {
            return null;
        }
        
        return $this->createUserEntity($user);
    }

    private function createUserEntity(User $user): PublicKeyCredentialUserEntity
    {
        //We create a PublicKeyCredentialUserEntity object
        // This object requires the username, the ID and the name to display (e.g. "John Doe")
        // The avatar URL is optionnal and could be null
        return new PublicKeyCredentialUserEntity(
            $user->username,
            $user->id,
            $user->displayName,
            $user->avatarUrl
        );
    }
}
```
{% endcode %}
