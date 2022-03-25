# Register Additional Authenticators

In some circumstances, you may need to register a new authenticator for a user e.g. when adding a new authenticator or when an administrator acts as another user to replace a lost device.

It is possible to perform this ceremony programmatically.

{% hint style="success" %}
You can attach several authenticators to a user account. It is recommended in case of lost devices or if the user gets access on your application using multiple platforms (smartphone, laptop…).
{% endhint %}

## The Easy Way

The procedure is the same as [the one described in this page](broken-reference), except that you don’t have to save the user entity again.

## The Hard Way

The procedure is the same as [the one described in this page](../the-webauthn-server/the-hard-way/authenticator-registration.md).

## The Symfony Way

{% hint style="info" %}
The following procedure is only available with the version 3.3.0+ of the bundle.
{% endhint %}

With a Symfony application, the fastest way for a user to register additional authenticators is to use the “controller” feature.

To add a new authenticator to a user, the bundle needs to know to whom it should be added. This can be:

* The current user itself e.g. from its own account
* An administrator acting for another user from a dashboard

For that purpose, a User Entity Guesser service should be created. This servuce shall implement the interface `Webauthn\Bundle\Security\Guesser\UserEntityGuesser` and its unique method `findUserEntity`. In the example herafter, the current user is used as User Entity.

```php
<?php

declare(strict_types=1);

namespace App\Service;

use Assert\Assertion;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Security\Core\Security;
use Webauthn\Bundle\Security\Guesser\UserEntityGuesser;
use Webauthn\PublicKeyCredentialUserEntity;

final class CurrentUserEntityGuesser implements UserEntityGuesser
{
    /**
     * @var Security
     */
    private $security;

    public function __construct(Security $security)
    {
        $this->security = $security;
    }

    public function findUserEntity(Request $request): PublicKeyCredentialUserEntity
    {
        $user = $this->security->getUser();
        Assertion::isInstanceOf($user, PublicKeyCredentialUserEntity::class, 'Unable to find the user entity');

        return $user;
    }
}
```

{% hint style="info" %}
In the case the current user is an administrator, the user entity can be determined using query parameters e.g. using routes like `/admin/add-authenticator/for/{USER_ID}`.

The user is retrieved using the associated repository and the given ID.
{% endhint %}

Now you just have to enable the feature and set the routes to your options and response controllers.

```yaml
webauthn:
    controllers:
        enabled: true # We enable the feature
        creation:
            from_user_account: # Unique name of our endpoints
                options_path: '/profile/security/devices/add/options' # Path to the creation options controller
                result_path: '/profile/security/devices/add' # Path to the response controller
                user_entity_guesser: App\Service\CurrentUserEntityGuesser # See above
            from_admin_dashboard: # Unique name of our endpoints
                options_path: '/admin/security/user/{USER_ID}/devices/add/options' # Path to the creation options controller
                result_path: '/admin/security/user/{USER_ID}/devices/add' # Path to the response controller
                user_entity_guesser: App\Service\AdminGuesser # Fictive service
```

{% hint style="warning" %}
As the user shall be authenticated to register a new authenticator, you should protect these routes in the `security.yaml` file.
{% endhint %}

{% code title="config/packages/security.yaml" %}
```yaml
security:
    access_control:
        - { path: ^/profile,  roles: IS_AUTHENTICATED_FULLY } # We protect all the /profile path
        - { path: ^/admin,  roles: ROLE_ADMIN }
```
{% endcode %}

Now you can send requests to these new endpoints. For example, if you are using the Javascript library, the calls will look like as follow:

```javascript
// Import the registration hook
import {useRegistration} from 'webauthn-helper';

// Create your register function.
// By default the urls are "/register" and "/register/options"
// but you can change those urls if needed.
const register = useRegistration({
    actionUrl: '/profile/security/devices/add',
    optionsUrl: '/profile/security/devices/add/options'
});


// We can call this register function whenever we need
//   No "username" or "displayName" parameters are needed
//   as the user entity is guessed by the dedicated service
register({})
    .then((response) => console.log('Registration success'))
    .catch((error) => console.log('Registration failure'))
;
```

### Creation Profile

The `default` [creation profile](../the-webauthn-server/the-symfony-way/#creation-profiles) is used. You can change it using the dedicated option.

```yaml
webauthn:
    controllers:
        enabled: true
        creation:
            from_user_account:
                …
                profile: custom_profile
```

### Response Handlers

You can customize the responses returned by the controllers by using custom handlers. This could be useful when you want to return additional information to your application.

There are 3 types of responses and handlers:

* Creation options,
* Success,
* Failure.

#### Creation Options Handler

This handler is called during the registration of a authenticator and has to implement the interface `Webauthn\Bundle\Security\Handler\CreationOptionsHandler`.

```yaml
webauthn:
    controllers:
        enabled: true
        creation:
            from_user_account:
                …
                options_handler: … # Your handler here
```

#### Success Handler

This handler is called when a client sends a valid assertion from the authenticator. This handler shall implement the interface `Webauthn\Bundle\Security\Handler\SuccessHandler`. The default handler is `Webauthn\Bundle\Service\DefaultSuccessHandler`.

```yaml
webauthn:
    controllers:
        enabled: true
        creation:
            from_user_account:
                …
                success_handler: … # Your handler here
```

#### Failure Handler

This handler is called when an error occurred during the process. This handler shall implement the interface `Webauthn\Bundle\Security\Handler\SuccessHandler`. The default handler is `Webauthn\Bundle\Service\DefaultFailureHandler`.

```yaml
webauthn:
    controllers:
        enabled: true
        creation:
            from_user_account:
                …
                failure_handler: … # Your handler here
```
