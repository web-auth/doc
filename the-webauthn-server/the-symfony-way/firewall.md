---
description: How to register and authenticate my users?
---

# Firewall

## Security Bundle

To authenticate or register your users with Symfony, the best and easiest way is to use the Security Bundle. First, install that bundle and follow the instructions given by [the official documentation](https://symfony.com/doc/current/security.html).

At the end of the installation and configuration, you should have a `config/packages/security.yaml` file that looks like as follow:

{% code title="config/packages/security.yaml" %}
```yaml
security:
    providers:
        default:
            id: App\Security\UserProvider
    firewalls:
        main:
            logout:
                path: 'logout'
            ...
```
{% endcode %}

## User Authentication

To enable the user authentication, you just have to declare the webauthn authenticator if the appropriate firewall (here `main`).

{% code title="config/packages/security.yaml" %}
```yaml
security:
    providers:
        default:
            id: App\Security\UserProvider
    firewalls:
        main:
            webauthn: ~
            logout:
                path: '/logout'
    access_control:
        - { path: ^/logout,  roles: PUBLIC_ACCESS}
```
{% endcode %}

As you have noticed, there is nothing to configure to have a fully functional firewall. The firewall routes are automatically created for you. They are namely:

* `/login/options`: to create the request options (`POST` only)
* `/login`: to submit the assertion response (`POST` only)

{% hint style="info" %}
Note that the endpoint shall be secured (`https`)
{% endhint %}

You should also ensure to allow anonymous users to contact those endpoints.

```yaml
security:
    firewalls:
        main:
            webauthn: ~
            logout:
                path: '/logout'

    access_control:
        - { path: ^/logout,  roles: PUBLIC_ACCESS}
        - { path: ^/login,  roles: PUBLIC_ACCESS, requires_channel: 'https'}
```

If you need, you can customize those endpoints (do not forget to allow anonymous access).

```yaml
security:
    firewalls:
        main:
            webauthn:
                authentication:
                    routes:
                        options_path: '/assertion/options'
                        result_path: '/assertion/result'

    access_control:
        - { path: ^/assertion,  roles: PUBLIC_ACCESS}
```

### Request Profile

By default, the `default` profile is used (see `request_profiles` in the [Configuration References](configuration-references.md)). You may have created a request profile in the bundle configuration. You can use this profile instead of the default one.

```yaml
security:
    firewalls:
        main:
            webauthn:
                authentication:
                    profile: 'acme'
```

## User Registration

The user registration can also managed by the firewall. It is disabled by default. If you want that feature, please enable it:

```yaml
security:
    firewalls:
        main:
            webauthn:
                registration:
                    enabled: true
```

The firewall routes are automatically created for you. They are namely:

* `/register/options`: to create the creation options (POST only)
* `/register`: to submit the attestation response (POST only)

You should also ensure to allow anonymous users to contact those endpoints.

```yaml
security:
    access_control:
        - { path: ^/register,  roles: PUBLIC_ACCESS}
```

## Authentication Attributes

The security token returned by the firewall sets some attributes depending on the assertion and the capabilities of the authenticator. The attributes are:

* `IS_USER_PRESENT`: the user was present during the authentication ceremony. This attribute is usually set to `true` by authenticators,
* `IS_USER_VERIFIED`: the user was verified by the authenticator. Verification may be performed by several means including biometrics ones (fingerprint, iris, facial recognition…).

You can then set constraints to the access controls. In the example below, the /admin path can be reached by users with the role `ROLE_ADMIN` and that **have been verified** during the ceremony.

```yaml
security:
    access_control:
        - { path: ^/admin,  roles: [ROLE_ADMIN, IS_USER_VERIFIED]}
```

## Response Handlers

You can customize the responses returned by the firewall by using a custom handler. This could be useful when you want to return additional information to your application.

There are 4 types of responses and handlers:

* Request options: options returned during the authentication ceremony,
* Creation options: options returned during the registration ceremony,
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

### Authentication Failure Handler

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
