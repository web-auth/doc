---
description: Step-by-step guide for migrating from 5.x to 6.0
---

# From 5.x to 6.0

{% hint style="info" %}
This page is subject to changes as the version 6.0.0 is not available at the time of writing.
{% endhint %}

This project follows the [Semantic Versioning principles](https://semver.org) and, contrary to upgrading a minor version (where the middle number changes) where no difficulty should be encountered, upgrading a major version (where the first number changes) is subject to significant modifications.

## Deprecations

### PublicKeyCredentialEntity.icon

`PublicKeyCredentialEntity.icon` is deprecated since `5.1.0`. This property is removed from the specification and is not used anymore.

### PublicKeyCredentialRpEntity.name

{% hint style="warning" %}
**Deprecated in v5.3.0**
{% endhint %}

The `name` property of `PublicKeyCredentialRpEntity` is deprecated in version 5.3.0 and will be removed in version 6.0.0. According to the WebAuthn Level 3 specification, the Relying Party name is no longer required.

```php
# Before (deprecated)
$rpEntity = PublicKeyCredentialRpEntity::create(
    name: 'My Application',
    id: 'example.com'
);

# After
$rpEntity = PublicKeyCredentialRpEntity::create(
    id: 'example.com'
);
```

### PublicKeyCredentialSource

{% hint style="warning" %}
**Deprecated in v5.3.0**
{% endhint %}

The class `Webauthn\PublicKeyCredentialSource` has been renamed to `Webauthn\CredentialRecord` to better reflect its purpose. The old class now extends `CredentialRecord` for backward compatibility but will be removed in version 6.0.0.

```php
# Before (deprecated)
use Webauthn\PublicKeyCredentialSource;

$credential = new PublicKeyCredentialSource(/* ... */);

# After
use Webauthn\CredentialRecord;

$credential = new CredentialRecord(/* ... */);
```

Similarly, the repository interface `PublicKeyCredentialSourceRepositoryInterface` is deprecated in favor of `CredentialRecordRepositoryInterface`.

### DoctrineCredentialSourceRepository

{% hint style="warning" %}
**Deprecated in v5.2.0**
{% endhint %}

The class `Webauthn\Bundle\Repository\DoctrineCredentialSourceRepository` provided by the Symfony bundle is deprecated and will be removed in version 6.0.0. You should create your own Doctrine-based repository instead.

```php
# Before (deprecated)
use Webauthn\Bundle\Repository\DoctrineCredentialSourceRepository;

# After — create your own repository
use Doctrine\Bundle\DoctrineBundle\Repository\ServiceEntityRepository;
use Doctrine\Persistence\ManagerRegistry;
use Webauthn\Bundle\Repository\CredentialRecordRepositoryInterface;
use Webauthn\Bundle\Repository\CanSaveCredentialRecord;
use Webauthn\CredentialRecord;

class WebauthnCredentialRepository extends ServiceEntityRepository implements CredentialRecordRepositoryInterface, CanSaveCredentialRecord
{
    // Implement findOneByCredentialId(), findAllForUserEntity(), saveCredentialRecord()
}
```

See the [Credential Record Repository](../symfony-bundle/credential-record-repository.md) page for a complete implementation example.

### createFormJson

{% hint style="warning" %}
**Deprecated in v5.3.0**
{% endhint %}

The `createFormJson` method is deprecated and will be removed in version 6.0.0. Use the standard Symfony Serializer to deserialize credential responses instead.

### Composer package `web-auth/webauthn-stimulus`

{% hint style="warning" %}
**Deprecated in v5.3.0 — removed in v6.0.0**
{% endhint %}

The dedicated PHP package `web-auth/webauthn-stimulus` (the Symfony Flex/AssetMapper wrapper around the Stimulus controllers) is deprecated. The same JavaScript is now published to npm as [`@web-auth/webauthn-stimulus`](https://www.npmjs.com/package/@web-auth/webauthn-stimulus) and that is the only package that will keep being maintained in 6.0.0.

Migrate your application before upgrading to 6.0.0:

```bash
# Before (deprecated)
composer remove web-auth/webauthn-stimulus

# After — pin from npm via AssetMapper
php bin/console importmap:require @web-auth/webauthn-stimulus

# Or with any bundler (Webpack Encore, Vite, esbuild…)
npm install @web-auth/webauthn-stimulus
```

The Stimulus controller names (`@web-auth/webauthn-stimulus`, `@web-auth/webauthn-stimulus/authentication`, `@web-auth/webauthn-stimulus/registration`) are unchanged, so your Twig templates do not need to be modified.

### Authenticator Transport CABLE

{% hint style="warning" %}
**Deprecated in v5.3.0**
{% endhint %}

The constant `AUTHENTICATOR_TRANSPORT_CABLE` is deprecated and will be removed in version 6.0.0. Use `AUTHENTICATOR_TRANSPORT_HYBRID` (the spec-aligned successor for caBLE / cloud-assisted BLE) instead.

```php
# Before (deprecated)
use Webauthn\PublicKeyCredentialDescriptor;

$transport = PublicKeyCredentialDescriptor::AUTHENTICATOR_TRANSPORT_CABLE;

# After
use Webauthn\PublicKeyCredentialDescriptor;

$transport = PublicKeyCredentialDescriptor::AUTHENTICATOR_TRANSPORT_HYBRID;
```

### New Authenticator Transports

{% hint style="info" %}
**Added in v5.3.0**
{% endhint %}

`PublicKeyCredentialDescriptor` exposes two new transport constants in addition to the historic `usb`, `nfc`, `ble` and `internal`:

- `AUTHENTICATOR_TRANSPORT_SMART_CARD` (`smart-card`)
- `AUTHENTICATOR_TRANSPORT_HYBRID` (`hybrid`, replaces `cable`)

All seven values are referenced by `PublicKeyCredentialDescriptor::AUTHENTICATOR_TRANSPORTS`.

### Options Handlers Signature

{% hint style="warning" %}
**Changed in v5.3.0**
{% endhint %}

The `CreationOptionsHandler` and `RequestOptionsHandler` interfaces now accept an optional `?Request $request` parameter. If you implement these interfaces, you must update the signature of your methods.

```php
# Before
use Symfony\Component\HttpFoundation\Response;
use Webauthn\PublicKeyCredentialCreationOptions;
use Webauthn\PublicKeyCredentialUserEntity;

class MyCreationOptionsHandler implements CreationOptionsHandler
{
    public function onCreationOptions(
        PublicKeyCredentialCreationOptions $options,
        PublicKeyCredentialUserEntity $userEntity,
    ): Response {
        // ...
    }
}

# After
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Webauthn\PublicKeyCredentialCreationOptions;
use Webauthn\PublicKeyCredentialUserEntity;

class MyCreationOptionsHandler implements CreationOptionsHandler
{
    public function onCreationOptions(
        PublicKeyCredentialCreationOptions $options,
        PublicKeyCredentialUserEntity $userEntity,
        ?Request $request = null,
    ): Response {
        // ...
    }
}
```

The same applies to `RequestOptionsHandler::onRequestOptions()`.

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
