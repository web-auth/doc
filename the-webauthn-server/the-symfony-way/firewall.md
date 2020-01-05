# Firewall

To authenticate your users, you can follow the steps described [on the previous page](authenticate-your-users.md). But as Symfony offers a firewall, you may prefer to use it instead of writing a new authentication process and get all advantages of this firewall.

First of all, you must install the dedicated bundle: `symfony/security-bundle`.

Next, you must [create a request profile as showed here](authenticate-your-users.md).

Then, you must have a PSR-7 message factory service. We recommend the use of `nyholm/psr7`, but feel free to use any other compatible library.

{% hint style="info" %}
The PSR-7 Message Factory shall be available as a service.
{% endhint %}

```yaml
security:
    firewalls:
        main:
            webauthn_json:
                profile: 'acme'
                http_message_factory: 'Nyholm\Psr7\Factory\Psr17Factory'
```

That's it! You can now protect any route as usual.

```yaml
security:
    access_control:
        - { path: ^/login,  roles: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/admin,  roles: 'ROLE_ADMIN' }
        - { path: ^/page,   roles: 'ROLE_USER' }
        - { path: ^/,       roles: IS_AUTHENTICATED_ANONYMOUSLY }
```

### Credential Request Options

Prior to the authentication of the user, you must get a PublicKey Credential Request Options object. To do so, send a POST request to `/login/options`.

The body of this request is a JSON object that must contain a `username` member with the name of the user being authenticated.

{% hint style="warning" %}
It is mandatory to set the Content-Type header to `application/json`.
{% endhint %}

#### Example

```javascript
fetch('/login/options', {
    method  : 'POST',
    credentials : 'same-origin',
    headers : {
        'Content-Type' : 'application/json'
    },
    body: JSON.stringify({
        "username": "john.doe"
    })
}).then(function (response) {
    return response.json();
}).then(function (json) {
    console.log(json);
}).catch(function (err) {
    console.log({ 'status': 'failed', 'error': err });
})
```

In case of success, you receive a valid `PublicKeyCredentialRequestOptions`  object and your user will be asked to interact with its security devices.

The default path is `/login/options`. You can change it if needed:

```yaml
security:
    firewalls:
        main:
            webauthn_json:
                options_path: /security/authentication/options
    access_control:
        - { path: ^/security,  roles: IS_AUTHENTICATED_ANONYMOUSLY}
```

### User Assertion

When the user touched the security device, you will receive a response from it. You just have to send a POST request to `/login`.

The body of this request is the response of the security device.

{% hint style="warning" %}
It is mandatory to set the Content-Type header to `application/json`.
{% endhint %}

#### Example:

```javascript
fetch('/assertion/result', {
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

The default path is `/assertion/result`. You can change that path is needed:

```yaml
security:
    firewalls:
        main:
            webauthn_json:
                login_path: /security/authentication/login
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

### Handlers

You can customize the responses returned by the firewall by using a custom handler. This could be useful when using an access token manager \(e.g. [LexikJWTAuthenticationBundle](https://github.com/lexik/LexikJWTAuthenticationBundle)\) or to modify the responses.

There are 3 types of responses and handlers:

* Request options,
* Authentication Success,
* Authentication Failure,

#### Request Options Handler

This handler is called when a client sends a valid POST request to the `options_path`. The default Request Options Handler is `Webauthn\Bundle\Security\Handler\DefaultRequestOptionsHandler`. It returns a JSON Response with the Public Key Credential Request Options objects in its body.

Your custom handler have to implement the interface `Webauthn\Bundle\Security\Handler\RequestOptionsHandler` and be declared as a service.

When done, you can set your new service in the firewall configuration:

```yaml
security:
    firewalls:
        main:
            webauthn_json:
                request_options_handler: 'App\Handler\MyCustomRequestOptionsHandler'
```

#### Authentication Success Handler

This handler is called when a client sends a valid assertion from the authenticator. The default handler is `Webauthn\Bundle\Security\Handler\DefaultSuccessHandler`.

Your custom handler have to implement the interface `Symfony\Component\Security\Http\Authentication\AuthenticationSuccessHandlerInterface` and be declared as a container service.

When done, you can set your new service in the firewall configuration:

```yaml
security:
    firewalls:
        main:
            webauthn_json:
                success_handler: 'App\Handler\MyCustomAuthenticationSuccessHandler'
```

#### Authentication Failure Handler

This handler is called when an error occurred during the authentication process. The default handler is `Webauthn\Bundle\Security\Handler\DefaultFailureHandler`.

Your custom handler have to implement the interface `Symfony\Component\Security\Http\Authentication\AuthenticationFailureHandlerInterface` and be declared as a container service.

When done, you can set your new service in the firewall configuration:

```yaml
security:
    firewalls:
        main:
            webauthn_json:
                failure_handler: 'App\Handler\MyCustomAuthenticationFailureHandler'
```

### Request Options Storage

Webauthn authentication is a 2 steps round trip authentication:

* Request options issuance
* Authenticator assertion verification

It is needed to store the request options and the user entity associated to it to verify the authenticator assertions.

By default, the firewall uses `Webauthn\Bundle\Security\Storage\SessionStorage`. This storage system stores the data in a session.

If this behaviour does not fit on your needs \(e.g. you want to use a database, REDIS…\), you can implement a custom data storage for that purpose. Your custom storage system have to implement `Webauthn\Bundle\Security\Storage\RequestOptionsStorage` and declared as a container service.

When done, you can set your new service in the firewall configuration:

```yaml
security:
    firewalls:
        main:
            webauthn_json:
                request_options_storage: 'App\Handler\MyCustomRequestOptionsStorage'
```

### Authentication Attributes

The security token returned by the firewall sets some attributes depending on the assertion and the capabilities of the authenticator. The attributes are:

* `IS_USER_PRESENT`: the user was present during the authentication ceremony. This attribute is usually set to `true` by Webauthn authenticators,
* `IS_USER_VERIFIED`: the user was verified by the authenticator. Verification may be performed by several means including biometrics ones \(fingerprint, iris, facial recognition…\).

You can then set constraints to the access controls.

```yaml
security:
    access_control:
        - { path: ^/admin,  roles: IS_USER_VERIFIED}
```

