---
description: Step-by-step guide for migrating from 5.x to 6.0
---

# From 5.x to 6.0

{% hint style="info" %}
This page is subject to changes as the version 6.0.0 is not available at the time of writing.
{% endhint %}

This project follows the [Semantic Versioning principles](https://semver.org) and, contrary to upgrading a minor version (where the middle number changes) where no difficulty should be encountered, upgrading a major version (where the first number changes) is subject to significant modifications.

## What does an application look like in 6.0?

The biggest shift in 5.4 is the move from a **profile-driven** YAML configuration to **autowired helpers** invoked from controllers you write yourself. Once 6.0 lands, a typical Symfony application looks like this:

{% code title="config/packages/webauthn.yaml" %}
```yaml
webauthn:
    credential_repository: 'App\Repo\CredentialRepository'
    user_repository:       'App\Repo\UserRepository'
    options_storage:       'App\Storage\SessionStorage'

    metadata:
        enabled: true
        mds_repository:           'App\Webauthn\MetadataStatementRepository'
        status_report_repository: 'App\Webauthn\StatusReportRepository'
```
{% endcode %}

{% code title="config/services.yaml (optional)" %}
```yaml
parameters:
    app.webauthn.origins:
        - 'https://app.example.com'
        - 'https://admin.example.com'
```
{% endcode %}

Four user-written controllers cover the request and response sides of registration and authentication, calling the autowired `WebauthnOptionsResponse` and `WebauthnResponseVerifier` helpers. See the [Options Helpers](../symfony-bundle/options-helpers.md) and [Verification Helpers](../symfony-bundle/verification-helpers.md) pages for the concrete patterns.

| Concern | 5.3 (config-driven) | 6.0 (helper-driven) |
|---|---|---|
| Routes | `controllers.creation[].options_path`, `result_path` | `#[Route]` attributes on user controllers |
| Profile (challenge length, RP, UV, attestation, etc.) | `creation_profiles` / `request_profiles` | `with*()` setters on the builder returned by `forCreation()` / `forRequest()` |
| Allowed origins | `allowed_origins` (root + per-controller) | `withAllowedOrigins(...)` on the verifier (or omit for the W3C same-origin fallback on single-domain apps) |
| Client overrides | `client_override_policy` (array-in-array YAML) | `ClientOverridePolicy` built inline + `withClientOverrides()` |
| Anti-enumeration | implicit, via `fake_credential_generator` service ID | active by default on `forRequest()`; `withFakeCredentialGenerator(null)` opts out |
| User entity guesser | `controllers.creation[].user_entity_guesser` | second positional argument of `forCreation($rpId, $guesser)` |
| Conditional Create | `creation_profiles[].conditional_create: true` | `withMediation('conditional')` on the creation builder; the verifier auto-detects from the stored options |

## Deprecations

### Profile-driven configuration superseded by helpers

{% hint style="warning" %}
**Deprecated in v5.4.0**
{% endhint %}

The bundle's profile- and controller-driven YAML sections are superseded by the autowired helpers introduced in 5.4. Every section keeps working until 6.0; this section lists the migration path for each.

#### `webauthn.creation_profiles` / `webauthn.request_profiles`

Move to the `WebauthnOptionsResponse` helper from a controller of your own.

```yaml
# Before (deprecated)
webauthn:
    creation_profiles:
        default:
            rp:
                id: 'example.com'
            challenge_length: 32
            authenticator_selection_criteria:
                user_verification: 'preferred'
                resident_key: 'preferred'
            attestation_conveyance: 'none'
    request_profiles:
        default:
            rp_id: 'example.com'
            user_verification: 'preferred'
```

```php
// After
use Webauthn\AuthenticatorSelectionCriteria;
use Webauthn\Bundle\Service\WebauthnOptionsResponse;

#[Route('/webauthn/register/options', methods: ['POST'])]
public function __invoke(Request $request): JsonResponse
{
    return $this->options
        ->forCreation('example.com', $this->newUserGuesser)
        ->withChallengeLength(32)
        ->withAuthenticatorSelectionCriteria(
            AuthenticatorSelectionCriteria::create(
                userVerification: AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED,
                residentKey: AuthenticatorSelectionCriteria::RESIDENT_KEY_REQUIREMENT_PREFERRED,
            )
        )
        ->build($request);
}
```

#### `webauthn.controllers`

Replace each `controllers.creation[name]` and `controllers.request[name]` block by a pair of user controllers (one for the options endpoint, one for the response endpoint). Routes move to `#[Route]` attributes; the per-controller `host`, `allowed_origins`, `allow_subdomains`, `hide_existing_credentials`, `user_entity_guesser` etc. all have a direct `with*()` equivalent on the helper.

The full refactored example is on the [Verification Helpers](../symfony-bundle/verification-helpers.md) page.

#### `webauthn.client_override_policy`

Build a `ClientOverridePolicy` inline in the controller and attach it to the helper. 5.4 ships a typed `ClientOverrideRule` value object that makes the call site much clearer than the legacy nested-array form (both shapes stay supported as first-class APIs):

```yaml
# Before (deprecated)
webauthn:
    client_override_policy:
        user_verification:
            enabled: true
            allowed_values: [preferred, required]
```

```php
// After (5.4) — typed factory recommended
use Webauthn\Bundle\Policy\ClientOverridePolicy;
use Webauthn\Bundle\Policy\ClientOverrideRule;

return $this->options
    ->forRequest('example.com')
    ->withClientOverrides(ClientOverridePolicy::fromRules(
        userVerification: ClientOverrideRule::restrictTo(['preferred', 'required']),
    ))
    ->build($request);
```

```php
// Or the legacy nested-array form, also first-class
return $this->options
    ->forRequest('example.com')
    ->withClientOverrides(new ClientOverridePolicy([
        'user_verification' => [
            'enabled' => true,
            'allowed_values' => ['preferred', 'required'],
        ],
    ]))
    ->build($request);
```

#### `webauthn.allowed_origins` / `webauthn.allow_subdomains`

Two migration paths depending on your topology.

**Single-origin app (most common)**: drop the YAML node entirely. The verifier falls back to the W3C-recommended same-origin check (`CheckOrigin` against the request host). Nothing else to do.

**Multi-origin app**: spread a Symfony parameter into the helper.

```yaml
# Before (deprecated)
webauthn:
    allowed_origins:
        - 'https://app.example.com'
        - 'https://admin.example.com'
    allow_subdomains: true
```

```yaml
# After — config/services.yaml
parameters:
    app.webauthn.origins:
        - 'https://app.example.com'
        - 'https://admin.example.com'
```

```php
// After — every verification controller
use Symfony\Component\DependencyInjection\Attribute\Autowire;

public function __construct(
    private readonly WebauthnResponseVerifier $verifier,
    #[Autowire('%app.webauthn.origins%')]
    private readonly array $origins,
) {}

#[Route('/webauthn/register', methods: ['POST'])]
public function __invoke(Request $request): Response
{
    $result = $this->verifier
        ->forAttestation('example.com')
        ->withAllowedOrigins(...$this->origins)
        ->withAllowSubdomains(true)
        ->verify($request);
    // ...
}
```

The list lives in a single Symfony parameter, the security-critical decision is visible right where it applies, and per-route overrides become trivial.

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
# 1. Drop the Composer wrapper
composer remove web-auth/webauthn-stimulus

# 2. Pin the npm package — pick one
php bin/console importmap:require @web-auth/webauthn-stimulus   # AssetMapper
npm install @web-auth/webauthn-stimulus                          # Encore / Vite / esbuild
```

Then register the controllers from your Stimulus bootstrap file (`assets/bootstrap.js` with the default AssetMapper recipe) under their package-prefixed identifiers:

```javascript
import { Application } from '@hotwired/stimulus';
import {
    AuthenticationController,
    RegistrationController,
    WebauthnController,
} from '@web-auth/webauthn-stimulus';

const app = Application.start();
app.register('web-auth--webauthn-stimulus--authentication', AuthenticationController);
app.register('web-auth--webauthn-stimulus--registration', RegistrationController);
app.register('web-auth--webauthn-stimulus', WebauthnController);
```

{% hint style="danger" %}
**Do not add `@web-auth/webauthn-stimulus` to `assets/controllers.json`.** Symfony UX `StimulusBundle` resolves `controllers.json` entries against installed Composer packages, so it throws `Could not find package "web-auth/webauthn-stimulus" referred to from controllers.json` once the Composer wrapper is gone. Register from JavaScript instead, as shown above.
{% endhint %}

Your Twig templates do not need any change — `stimulus_controller('@web-auth/webauthn-stimulus/authentication')` still resolves to the `web-auth--webauthn-stimulus--authentication` identifier you just registered.

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
