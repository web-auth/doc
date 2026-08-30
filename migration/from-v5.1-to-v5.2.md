---
description: Upgrade guide from 5.1 to 5.2
---

# From 5.1 to 5.2

{% hint style="success" %}
No backward compatibility break. Everything listed below keeps working until 6.0.0.
{% endhint %}

Version 5.2 reworks the origin verification, moves the options storage to a single configuration node and introduces the dedicated Symfony Passport and Badge, which supersede the `webauthn` firewall.

## Deprecations

### Secured RP IDs

{% hint style="warning" %}
**Deprecated in v5.2.0**
{% endhint %}

`secured_rp_ids` listed the Relying Party IDs allowed to be served over plain HTTP. It is superseded by `allowed_origins` and `allow_subdomains`, which check the full origin (scheme, host and port) instead of the sole host.

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

The node is deprecated at the three levels where it was available: the root configuration, the controllers and the firewall.

{% hint style="danger" %}
List only the origins that share the trust level of the endpoint, and never enable `allow_subdomains` when a subdomain can be controlled by a third party.
{% endhint %}

Without the Symfony bundle, the same move applies to the ceremony step manager:

```php
# Before (deprecated)
$factory->setSecuredRelyingPartyId(['localhost']);

# After
$factory->setAllowedOrigins(['http://localhost', 'https://example.com'], false);
```

The `Webauthn\CeremonyStep\CheckOrigin` step is deprecated as well and is replaced by `Webauthn\CeremonyStep\CheckAllowedOrigins`. The new step also accepts non-URL facet identifiers such as `android:apk-key-hash:...`, so a native Android application can share the Relying Party ID of your website.

### Options Storage

{% hint style="warning" %}
**Deprecated in v5.2.0**
{% endhint %}

The `options_storage` option at the controller and firewall levels is deprecated. Declare the storage once, at the root of the configuration.

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

The same node is deprecated on the firewall configuration. The `$optionStorage` argument of `AttestationControllerFactory` and `AssertionControllerFactory` follows the same path: pass `null` and rely on the global storage.

### DoctrineCredentialSourceRepository

{% hint style="warning" %}
**Deprecated in v5.2.0**
{% endhint %}

The class `Webauthn\Bundle\Repository\DoctrineCredentialSourceRepository` provided by the Symfony bundle is deprecated and will be removed in version 6.0.0. Create your own Doctrine-based repository instead.

```php
# Before (deprecated)
use Webauthn\Bundle\Repository\DoctrineCredentialSourceRepository;

# After: create your own repository
use Doctrine\Bundle\DoctrineBundle\Repository\ServiceEntityRepository;
use Doctrine\Persistence\ManagerRegistry;
use Webauthn\Bundle\Repository\CanSaveCredentialRecord;
use Webauthn\Bundle\Repository\CredentialRecordRepositoryInterface;
use Webauthn\CredentialRecord;

class WebauthnCredentialRepository extends ServiceEntityRepository implements CredentialRecordRepositoryInterface, CanSaveCredentialRecord
{
    // Implement findOneByCredentialId(), findAllForUserEntity(), saveCredentialRecord()
}
```

See the [Credential Record Repository](../symfony-bundle/credential-record-repository.md) page for a complete implementation example.

{% hint style="info" %}
The interface names above are the ones introduced in 5.3. On 5.2 they are still called `PublicKeyCredentialSourceRepositoryInterface` and `CanSaveCredentialSource`. See [From 5.2 to 5.3](from-v5.2-to-v5.3.md).
{% endhint %}

### Firewall

{% hint style="warning" %}
**Deprecated in v5.2.0**
{% endhint %}

Version 5.2 ships a dedicated `WebauthnPassport` and a `WebauthnBadge`, which make a custom authenticator straightforward. The `webauthn` firewall is deprecated and will be removed in 6.0.0.

#### The configuration

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

## Other changes

Nothing to do on your side, but worth knowing:

* the authentication extensions are reworked and the PRF extension is supported. See [Extensions](../pure-php/advanced-behaviours/extensions.md) and [PRF Extension](../pure-php/advanced-behaviours/prf-extension.md).
* the bundle supports CSRF protection and a remember-me checkbox at login.
* `android:apk-key-hash:...` facet identifiers are accepted in `allowed_origins`, so an Android application sharing your Relying Party ID passes the origin check.
