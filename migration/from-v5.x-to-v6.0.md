---
description: Step-by-step guide for migrating from 5.x to 6.0
---

# From 5.x to 6.0

{% hint style="info" %}
This page is subject to changes as the version 6.0.0 is not available at the time of writing.
{% endhint %}

This project follows the [Semantic Versioning principles](https://semver.org) and, contrary to upgrade a minor version (where the middle number changes) where no difficulty should be encountered, upgrade a major version (where the first number changes) is subject to significant modifications.

## Deprecations

### PublicKeyCredentialEntity.icon

`PublicKeyCredentialEntity.icon` is deprecated since `5.1.0` . This property is removed from the specification and is not used anymore.

### Secured RP IDs

`secured_rp_ids` is deprecated since `5.2.0`. Use `allowed_origins` and `allow_subdomains`.

```yaml
#Before
webauthn:
    secured_rp_ids:
      - 'localhost'
    controllers:
       enabled: true
       creation:
           test:
               hide_existing_credentials: true
               options_path: '/devices/add/options'
               result_path: '/devices/add'
               user_entity_guesser: 'Webauthn\Bundle\Security\Guesser\CurrentUserEntityGuesser'
               secured_rp_ids:
                 - 'bar.acme'

#After
webauthn:
    allowed_origins:
      - 'http://localhost'
      - 'https://bar.acme'
    allow_subdomains: false
```

### Options Storage

`options_storage` option on the controller or firewall levels are deprecated. Please use the top level configuration key

```yaml
#Before
webauthn:
  controllers:
    enabled: true
    creation:
      test:
        hide_existing_credentials: true
        options_path: '/devices/add/options'
        result_path: '/devices/add'
        user_entity_guesser: 'Webauthn\Bundle\Security\Guesser\CurrentUserEntityGuesser'
        options_storage: '...\CustomSessionStorage'

#After
webauthn:
  options_storage: '...\CustomSessionStorage'
  controllers:
    enabled: true
    creation:
      test:
        hide_existing_credentials: true
        options_path: '/devices/add/options'
        result_path: '/devices/add'
        user_entity_guesser: 'Webauthn\Bundle\Security\Guesser\CurrentUserEntityGuesser'
```

### Firewall

The `webauthn` firewall is deprecated. Please use the dedicated `Passport` and `Badge` instead.

#### The configuration:

```yaml
#Before
security:
  firewalls:
    main:
      webauthn:
        failure_handler: '...\FailureHandler'
        success_handler: '...\SuccessHandler'
        authentication:
          enabled: true
          routes:
            options_path: '/api/login/options'
            result_path: '/api/login'

#After
#config/packages/security.yaml
security:
  firewalls:
    main:
      custom_authenticator: 'App\Security\WebauthnAuthenticator' # See below
      
#config/packages/webauthn.yaml
webauthn:
    controllers:
        enabled: true
        request:
            login:
                options_path: '/login/webauthn/options'
```

#### The custom authenticator

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
use Webauthn\Bundle\Security\Authentication\WebauthnAuthenticator as BaseWebauthnAuthenticator;
use Webauthn\Bundle\Security\Authentication\WebauthnBadge;
use Webauthn\Bundle\Security\Authentication\WebauthnPassport;

final class WebauthnAuthenticator extends BaseWebauthnAuthenticator
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

#### The template

```twig
{% extends 'base.html.twig' %}

{% block body %}
    {% if error is defined %}
        <div>{{ error.messageKey|trans(error.messageData, 'security') }}</div>
    {% endif %}

    <form action="{{ path('app_login') }}" method="post">
        <input type="hidden" id="assertion" name="_assertion">
        <button id="login" name="login" type="submit">login</button>
    </form>
{% endblock %}
```

With the Stimulus Controller

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
                 requestOptionsUrl: path('webauthn.controller.request.request.login'),
                 requestResultField: 'input[name="_assertion"]',
             }
        ) }}
    >
        <input type="hidden" id="assertion" name="_assertion">
        <button id="login" name="login" type="submit" {{ stimulus_action('@web-auth/webauthn-stimulus', 'signin') }}>login</button>
    </form>
{% endblock %}
```
