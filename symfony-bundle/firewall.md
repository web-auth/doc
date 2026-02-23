---
description: How to register and authenticate my users?
---

# Firewall

## Security Bundle

To authenticate or register your users with Symfony, the best and easiest way is to use the Security Bundle. First, install that bundle and follow the instructions given by [the official documentation](https://symfony.com/doc/current/security.html).

At the end of the installation and configuration, you should have a `config/packages/security.yaml` file that looks like the following:

{% code title="config/packages/security.yaml" lineNumbers="true" %}
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

## Controller

If you are familiar with the Username/Password authentication with Symfony, it is not very different with WebAuthn.  You first need a controller to display the login form

```php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class LoginController extends AbstractController
{
    #[Route('/login', name: 'app_login')]
    public function index(AuthenticationUtils $authenticationUtils): Response
      {
         // get the login error if there is one
         $error = $authenticationUtils->getLastAuthenticationError();

         // last username entered by the user
         $lastUsername = $authenticationUtils->getLastUsername();

        return $this->render('login/index.html.twig', [
             'last_username' => $lastUsername,
             'error'         => $error,
          ]);
    }
}
```

## Template

Below is an example of a login form using the [Stimulus Controller](../symfony-ux/user-authentication.md). Also, please note that:

* The username field is not required
* The username field should have the attribute `autocomplete="username webauthn"`&#x20;

```twig
{% extends 'base.html.twig' %}

{% block body %}
    {% if error is defined %}
        <div>{{ error.messageKey|trans(error.messageData, 'security') }}</div>
    {% endif %}

    <form
        action="{{ path('app_login') }}"
        method="post"
        {{ stimulus_controller('@web-auth/webauthn-stimulus',
             {
                 useBrowserAutofill: true,
                 requestOptionsUrl: path('webauthn.controller.request.request.login'),
                 requestResultField: 'input[name="_assertion"]',
             }
        ) }}
    >
        <label for="username">Username:</label>
        <input type="text" id="username" name="_username" value="{{ last_username }}" placeholder="Type your username here" autocomplete="username webauthn">
        <input type="hidden" id="assertion" name="_assertion">
        <button id="login" name="login" type="submit">login</button>
    </form>
{% endblock %}
```

## Login Authenticator

Next, we need a Symfony Authenticator to handle login form submissions. With WebAuthn, we use a dedicated Passport that shall contain a specific Badge.

The WebAuthn Badge will receive the current host (i.e. the current domain) and the result from the FIDO2 Authenticator.

The WebAuthn Passport can receive any other badge you need e.g. the CSRF Token Badge or a custom badge required for your authentication flow.

```php
<?php

declare(strict_types=1);

namespace App\Security\Functional;

use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Generator\UrlGeneratorInterface;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Http\Authenticator\Passport\Passport;
use Webauthn\Bundle\Security\Authentication\WebauthnAuthenticator;
use Webauthn\Bundle\Security\Authentication\WebauthnBadge;
use Webauthn\Bundle\Security\Authentication\WebauthnPassport;

final class LoginAuthenticator extends WebauthnAuthenticator 
{
    public function __construct(
        private readonly UrlGeneratorInterface $urlGenerator,
    ) {
    }

    public function authenticate(Request $request): Passport
    {
        return new WebauthnPassport( #Dedicated Passport
            new WebauthnBadge( # Dedicated badge
                $request->getHost(),
                $request->request->get('_assertion', '') // From the login form. See below
            ),
            [/** Add other badges here */]
        );
    }

    public function onAuthenticationSuccess(Request $request, TokenInterface $token, string $firewallName): ?Response
    {
        return new JsonResponse([
            'success' => true,
        ]);
    }

    protected function getLoginUrl(Request $request): string
    {
        return $this->urlGenerator->generate('app_login'); //Redirect to the login controller
    }
}
```

## Security Configuration

To enable the user authentication, you just have to declare the Symfony Authenticator in the appropriate firewall (here `main`).

{% code title="config/packages/security.yaml" lineNumbers="true" %}
```yaml
security:
    providers:
        default:
            id: App\Security\UserProvider
    firewalls:
        main:
            custom_authenticator: 'App\Security\WebauthnAuthenticator'
            logout:
                path: '/logout'
    access_control:
        - { path: ^/login,  roles: PUBLIC_ACCESS, requires_channel: 'https'}
        - { path: ^/logout,  roles: PUBLIC_ACCESS}
```
{% endcode %}

{% hint style="info" %}
**Multiple User Providers (v5.2.1+):** The bundle now supports multiple user providers. If your application uses multiple providers, you can configure them in the security configuration and the WebauthnBadge will work correctly with all of them.
{% endhint %}

Also, you need to define the credential options endpoint.

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    controllers:
        enabled: true
        request:
            login:
                options_path: '/login/webauthn/options'
```
{% endcode %}

## User Registration

TO BE WRITTEN

## Authentication Attributes

The security token returned by the firewall sets some attributes depending on the assertion and the capabilities of the authenticator. The attributes are:

* `IS_USER_PRESENT`: the user was present during the authentication ceremony. This attribute is usually set to `true` by authenticators,
* `IS_USER_VERIFIED`: the user was verified by the authenticator. Verification may be performed by several means including biometrics ones (fingerprint, iris, facial recognition…).

You can then set constraints to the access controls. In the example below, the /admin path can be reached by users with the role `ROLE_ADMIN` and that **have been verified** during the ceremony.

{% code title="config/packages/security.yaml" lineNumbers="true" %}
```yaml
security:
    access_control:
        - { path: ^/admin,  roles: [ROLE_ADMIN, IS_USER_VERIFIED]}
```
{% endcode %}
