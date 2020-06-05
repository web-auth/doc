---
description: How to register and authenticate my users?
---

# Firewall

## Security Bundle

To authenticate or register your users with Symfony, the best and easiest way is to use the Security Bundle. First, install that bundle.

```bash
composer require symfony/security-bundle
```

Next, you have to create a custom [user provider](https://symfony.com/doc/current/security/user_provider.html) that will retrieve users on login requests. The following example uses the [user repository showed on this page](entities-with-doctrine.md#the-repository-1).

{% code title="src/Repository/UserRepository.php" %}
```php
<?php

namespace App\Security;

use App\Entity\User;
use App\Repository\UserRepository;
use Symfony\Component\Security\Core\Exception\UnsupportedUserException;
use Symfony\Component\Security\Core\Exception\UsernameNotFoundException;
use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Security\Core\User\UserProviderInterface;

class UserProvider implements UserProviderInterface
{
    /**
     * @var UserRepository
     */
    private $userRepository;

    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    /**
     * @throws UsernameNotFoundException if the user is not found
     */
    public function loadUserByUsername($username): UserInterface
    {
        $user = $this->userRepository->find($username);
        if (null !== $user) {
            return $user;
        }

        throw new UsernameNotFoundException(sprintf('User with name "%s" does not exist', $username));
    }

    public function refreshUser(UserInterface $user): UserInterface
    {
        if (!$user instanceof User) {
            throw new UnsupportedUserException(sprintf('Invalid user class "%s".', get_class($user)));
        }

        return $this->loadUserByUsername($user->getUsername());
    }

    public function supportsClass($class): bool
    {
        return User::class === $class;
    }
}
```
{% endcode %}

### Firewall Configuration

Now you can tell Symfony to use your user provider and you enable the dedicated firewall

{% code title="config/packages/security.yaml" %}
```yaml
security:
    providers:
        main:
            id: 'App\Security\UserProvider'
    firewalls:
        main:
            webauthn: ~
```
{% endcode %}

In some case, you may have several user providers that are used by other parts of your application. This Webauthn bundle allow you to override the default user provider.

{% code title="config/packages/security.yaml" %}
```yaml
security:
    providers:
        backend_users:
            memory:
                users:
                    john_admin: { password: '$2y$13$jxGxc ... IuqDju', roles: ['ROLE_ADMIN'] }
                    jane_admin: { password: '$2y$13$PFi1I ... rGwXCZ', roles: ['ROLE_ADMIN', 'ROLE_SUPER_ADMIN'] }
    firewalls:
        backend:
            pattern: ^/backend/
            # ... Add your configuration here
            
        main:
            webauthn:
                user_provider: 'App\Security\UserProvider'
```
{% endcode %}

### Logout

Users can logout as usual. You just have to add [the logout configuration](https://symfony.com/doc/current/security.html#logging-out) under your firewall and add the route.

{% code title="config/packages/security.yaml" %}
```yaml
security:
    # ...

    firewalls:
        main:
            # ...
            logout: ~
```
{% endcode %}

## User Authentication

As you have noticed, there is nothing to configure to have a fully functional firewall. The firewall routes are automatically created for you. They are namely:

* `/login/options`: to get the request options
* `/login`: to submit the assertion response

You should also ensure to allow anonymous users to contact those endpoints.

```yaml
security:
    access_control:
        - { path: ^/login,  roles: IS_AUTHENTICATED_ANONYMOUSLY }
```

### Credential Request Options

Prior to the authentication of the user, you must get a PublicKey Credential Request Options object. To do so, send a POST request to `/login/options`.

The body of this request is a JSON object that must contain a `username` field with the name of the user being authenticated.

{% hint style="success" %}
No need to reinvent the wheel, you can use the [webauthn-helper](https://www.npmjs.com/package/webauthn-helper) package.
{% endhint %}

{% hint style="warning" %}
It is mandatory to set the Content-Type header to `application/json`.
{% endhint %}

```javascript
fetch('/login/options', {
    method  : 'POST',
    credentials : 'same-origin',
    headers : {
        'Content-Type' : 'application/json'
    },
    body: JSON.stringify({
        "username": "johndoe@example.com"
    })
}).then(function (response) {
    return response.json();
}).then(function (json) {
    console.log(json);
}).catch(function (err) {
    console.log({ 'status': 'failed', 'error': err });
})
```

In case of success, you receive a valid `PublicKeyCredentialRequestOptions` object and your user will be asked to interact with its security devices.

The default path is `/login/options`. You can change it if needed:

```yaml
security:
    firewalls:
        main:
            webauthn:
                authentication:
                    routes:
                        options_path: /security/authentication/options
    access_control:
        - { path: ^/security/authentication/options,  roles: IS_AUTHENTICATED_ANONYMOUSLY}
```

### Assertion Response

When the user touched the security device, you will receive a response from it. You just have to send a POST request to `/login`.

The body of this request is the response of the security device.

{% hint style="warning" %}
It is mandatory to set the Content-Type header to `application/json`.
{% endhint %}

```javascript
fetch('/login', {
    method  : 'POST',
    credentials : 'same-origin',
    headers : {
        'Content-Type' : 'application/json'
    },
    body: //put the security device response here
}).then(function (response) {
    return response.json();
}).then(function (json) {
    console.log(json);
}).catch(function (err) {
    console.log({ 'status': 'failed', 'error': err });
})
```

The default path is `/login`. You can change that path if needed:

```yaml
security:
    firewalls:
        main:
            webauthn:
                authentication:
                    routes:
                        result_path: /security/authentication/login
```

Your user can now be authenticated and retrieved as usual.

{% code title="Acme\\Controller\\AdminController.php" %}
```php
<?php

declare(strict_types=1);

namespace Acme\Controller;

use Symfony\Component\Security\Core\Authentication\Token\Storage\TokenStorageInterface;

final class AdminController
{
    /**
     * @var TokenStorageInterface
     */
    private $tokenStorage:

    public function __construct(TokenStorageInterface $tokenStorage)
    {
        $this->tokenStorage = $tokenStorage;
    }

    public function __invoke()
    {
        // $token is an object of type Webauthn\Bundle\Security\Authentication\Token\WebauthnToken
        $token = $this->tokenStorage->getToken();
        ...
    }
}
```
{% endcode %}

### Request Profile

By default, the `default` profile is used. You may have created a request profile in the bundle configuration. You can use this profile instead of the default one.

```yaml
security:
    firewalls:
        main:
            webauthn:
                authentication:
                    profile: 'acme'
```

## User Registration

The user registration is also managed by the firewall. It is disabled by default. If you want that feature, please enable it:

```yaml
security:
    firewalls:
        main:
            webauthn:
                registration:
                    enabled: true
```

The firewall routes are automatically created for you. They are namely:

* `/register/options`: to get the creation options
* `/register`: to submit the attestation response

You should also ensure to allow anonymous users to contact those endpoints.

```yaml
security:
    access_control:
        - { path: ^/register,  roles: IS_AUTHENTICATED_ANONYMOUSLY }
```

### Credential Creation Options

Prior to the registration of a user and its authenticator, you must get a PublicKey Credential Creation Options object. To do so, send a POST request to `/register/options`.

The body of this request is a JSON object that must contain `username` and `displayName` fields with the username of the user being registered and the name displayed in the application.

{% hint style="warning" %}
It is mandatory to set the Content-Type header to `application/json`.
{% endhint %}

```javascript
fetch('/register/options', {
        method  : 'POST',
        credentials : 'same-origin',
        headers : {
            'Content-Type' : 'application/json'
        },
        body: JSON.stringify({
            "username": "johndoe@example.com",
            "displayName": "John Doe"
        })
    }).then(function (response) {
        return response.json();
    }).then(function (json) {
        console.log(json);
    }).catch(function (err) {
        console.log({ 'status': 'failed', 'error': err });
    })
```

In case of success, you receive a valid `PublicKeyCredentialCreationOptions` object and your user will be asked to interact with its security devices.

The default path is `/register/options`. You can change it if needed:

```yaml
security:
    firewalls:
        main:
            webauthn:
                registration:
                    routes:
                        options_path: /security/registration/options
    access_control:
        - { path: ^/security/registration/options,  roles: IS_AUTHENTICATED_ANONYMOUSLY}
```

### Attestation Response

When the user touched the security device, you will receive a response from it. You just have to send a POST request to `/register`.

The body of this request is the response of the security device.

{% hint style="warning" %}
It is mandatory to set the Content-Type header to `application/json`.
{% endhint %}

```javascript
fetch('/register', {
    method  : 'POST',
    credentials : 'same-origin',
    headers : {
        'Content-Type' : 'application/json'
    },
    body: //put the security device response here
}).then(function (response) {
    return response.json();
}).then(function (json) {
    console.log(json);
}).catch(function (err) {
    console.log({ 'status': 'failed', 'error': err });
})
```

The default path is `/register`. You can change that path is needed:

```yaml
security:
    firewalls:
        main:
            webauthn:
                registration:
                    routes:
                        result_path: /security/registration/result
```

In case of success, the user and the authenticator are correctly registered and automatically logged in.

### Creation Profile

By default, the `default` profile is used. You may have created a creation profile in the bundle configuration. You can use this profile instead of the default one.

```yaml
security:
    firewalls:
        main:
            webauthn:
                registration:
                    profile: 'acme'
```

## Authentication Attributes

The security token returned by the firewall sets some attributes depending on the assertion and the capabilities of the authenticator. The attributes are:

* `IS_USER_PRESENT`: the user was present during the authentication ceremony. This attribute is usually set to `true` by authenticators,
* `IS_USER_VERIFIED`: the user was verified by the authenticator. Verification may be performed by several means including biometrics ones \(fingerprint, iris, facial recognition…\).

You can then set constraints to the access controls. In the example below, the /admin path can be reached by users with the role `ROLE_ADMIN` and that **have been verified** during the ceremony.

```yaml
security:
    access_control:
        - { path: ^/admin,  roles: [ROLE_ADMIN, IS_USER_VERIFIED]}
```

## Options Storage

Webauthn authentication and registration are 2 steps round trip processes:

* Options issuance
* Authenticator response verification

It is needed to store the options and the user entity associated to it to verify the authenticator responses.

By default, the firewall uses `Webauthn\Bundle\Security\Storage\SessionStorage`. This storage system stores the data in a session.

If this behaviour does not fit on your needs \(e.g. you want to use a database, Redis…\), you can implement a custom data storage for that purpose. Your custom storage system has to implement `Webauthn\Bundle\Security\Storage\RequestOptionsStorage` and be declared as a container service.

When done, you can set your new service in the firewall configuration:

```yaml
security:
    firewalls:
        main:
            webauthn:
                options_storage: 'App\Handler\MyCustomRequestOptionsStorage'
```

## Response Handlers

You can customize the responses returned by the firewall by using a custom handler. This could be useful when you want to return additional information to your application.

There are 4 types of responses and handlers:

* Request options,
* Creation options,
* Authentication Success,
* Authentication Failure,

### Request Options Handler

This handler is called when a client sends a valid POST request to the `options_path` during the authentication process. The default Request Options Handler is `Webauthn\Bundle\Security\Handler\DefaultRequestOptionsHandler`. It returns a JSON Response with the Public Key Credential Request Options objects in its body.

Your custom handler has to implement the interface `Webauthn\Bundle\Security\Handler\RequestOptionsHandler` and be declared as a service.

When done, you can set your new service in the firewall configuration:

```yaml
security:
    firewalls:
        main:
            webauthn:
                authentication:
                    options_handler: 'App\Handler\MyCustomRequestOptionsHandler'
```

### Creation Options Handler

This handler is very similar to the previous one, except that it is called during the registration of a new user and has to implement the interface `Webauthn\Bundle\Security\Handler\CreationOptionsHandler`.

```yaml
ecurity:
    firewalls:
        main:
            webauthn:
                registration:
                    options_handler: 'App\Handler\MyCustomRequestOptionsHandler'
```

### Authentication Success Handler

This handler is called when a client sends a valid assertion from the authenticator. The default handler is `Webauthn\Bundle\Security\Handler\DefaultSuccessHandler`.

Your custom handler has to implement the interface `Symfony\Component\Security\Http\Authentication\AuthenticationSuccessHandlerInterface` and be declared as a container service.

When done, you can set your new service in the firewall configuration:

```yaml
security:
    firewalls:
        main:
            webauthn:
                success_handler: 'App\Handler\MyCustomAuthenticationSuccessHandler'
```

#### Authentication Failure Handler

This handler is called when an error occurred during the authentication process. The default handler is `Webauthn\Bundle\Security\Handler\DefaultFailureHandler`.

Your custom handler has to implement the interface `Symfony\Component\Security\Http\Authentication\AuthenticationFailureHandlerInterface` and be declared as a container service.

When done, you can set your new service in the firewall configuration:

```yaml
security:
    firewalls:
        main:
            webauthn:
                failure_handler: 'App\Handler\MyCustomAuthenticationFailureHandler'
```

