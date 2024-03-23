# Register Additional Authenticators

In some circumstances, you may need to register a new authenticator for a user e.g. when adding a new authenticator or when an administrator acts as another user to replace a lost device.

It is possible to perform this ceremony programmatically.

{% hint style="success" %}
You can attach several authenticators to a user account. It is recommended in case of lost devices or if the user gets access on your application using multiple platforms (smartphone, laptop…).
{% endhint %}

With a Symfony application, the fastest way for a user to register additional authenticators is to use the “controller” feature.

To add a new authenticator to a user, the bundle needs to know to whom it should be added. This can be:

* The current user itself e.g. from its own account
* An administrator acting for another user from a dashboard

For that purpose, a User Entity Guesser service should be created. This service shall implement the interface `Webauthn\Bundle\Security\Guesser\UserEntityGuesser` and its unique method `findUserEntity`.

You can directly use the `Webauthn\Bundle\Security\Guesser\CurrentUserEntityGuesser` as a Symfony service. It is designed to identify the user that is currently logged in.

In the example herafter where the current user is guessed using a controller parameter. This can be used when an administrator is adding an authenticator to another user account.

{% code title="App\Guesser\FromQueryParameterGuesser.php" lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

namespace App\Guesser;

use Assert\Assertion;
use Symfony\Component\HttpFoundation\Request;
use Webauthn\Bundle\Repository\PublicKeyCredentialUserEntityRepository;
use Webauthn\Bundle\Security\Guesser\UserEntityGuesser;
use Webauthn\PublicKeyCredentialUserEntity;

final class FromQueryParameterGuesser implements UserEntityGuesser
{
    public function __construct(
        private PublicKeyCredentialUserEntityRepository $userEntityRepository
    ) {
    }

    public function findUserEntity(Request $request): PublicKeyCredentialUserEntity
    {
        $userHandle = $request->query->get('user_id');
        Assertion::string($userHandle, 'User entity not found. Invalid user ID');
        $user = $this->userEntityRepository->findOneByUserHandle($userHandle);
        Assertion::isInstanceOf($user, PublicKeyCredentialUserEntity::class, 'User entity not found.');

        return $user;
    }
}
```
{% endcode %}

{% hint style="info" %}
In the case the current user s supposed to be administrator, the user entity can be determined using the query parameters and a route like `/admin/add-authenticator/for/{user_id}`.
{% endhint %}

Now you just have to enable the feature and set the routes to your options and response controllers.

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    controllers:
        enabled: true # We enable the feature
        creation:
            from_user_account: # Endpoints accessible by the user itself
                options_path: '/profile/security/devices/add/options' # Path to the creation options controller
                result_path: '/profile/security/devices/add' # Path to the response controller
                user_entity_guesser: Webauthn\Bundle\Security\Guesser\CurrentUserEntityGuesser # See above
            from_admin_dashboard: # Endpoint accessible by an administrator
                options_path: '/admin/security/user/{user_id}/devices/add/options' # Path to the creation options controller
                result_path: '/admin/security/user/{user_id}/devices/add' # Path to the response controller
                user_entity_guesser: App\Guesser\FromQueryParameterGuesser # From the example
```
{% endcode %}

{% hint style="warning" %}
As the user shall be authenticated to register a new authenticator, you should protect these routes in the `security.yaml` file.
{% endhint %}

{% code title="config/packages/security.yaml" lineNumbers="true" %}
```yaml
security:
    access_control:
        - { path: ^/profile,  roles: IS_AUTHENTICATED_FULLY } # We protect all the /profile path
        - { path: ^/admin,  roles: ROLE_ADMIN }
```
{% endcode %}

Now you can send requests to these new endpoints. For example, if you are using the Javascript library, the calls will look like as follow:

{% code lineNumbers="true" %}
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
{% endcode %}

### Creation Profile

The `default` [creation profile](../the-symfony-way.md#creation-profiles) is used. You can change it using the dedicated option.

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    controllers:
        enabled: true
        creation:
            from_user_account:
                …
                profile: custom_profile
```
{% endcode %}

### Response Handlers

You can customize the responses returned by the controllers by using custom handlers. This could be useful when you want to return additional information to your application.

There are 3 types of responses and handlers:

* Creation options,
* Success,
* Failure.

#### Creation Options Handler

This handler is called during the registration of a authenticator and has to implement the interface `Webauthn\Bundle\Security\Handler\CreationOptionsHandler`.

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    controllers:
        enabled: true
        creation:
            from_user_account:
                …
                options_handler: … # Your handler here
```
{% endcode %}

#### Success Handler

This handler is called when a client sends a valid assertion from the authenticator. This handler shall implement the interface `Webauthn\Bundle\Security\Handler\SuccessHandler`. The default handler is `Webauthn\Bundle\Service\DefaultSuccessHandler`.

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    controllers:
        enabled: true
        creation:
            from_user_account:
                …
                success_handler: … # Your handler here
```
{% endcode %}

#### Failure Handler

This handler is called when an error occurred during the process. This handler shall implement the interface `Webauthn\Bundle\Security\Handler\SuccessHandler`. The default handler is `Webauthn\Bundle\Service\DefaultFailureHandler`.

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    controllers:
        enabled: true
        creation:
            from_user_account:
                …
                failure_handler: … # Your handler here
```
{% endcode %}
