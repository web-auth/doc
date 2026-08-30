---
description: Step-by-step guide for migrating from 5.4 to 6.0
---

# From 5.4 to 6.0

{% hint style="info" %}
This page is subject to changes as the version 6.0.0 is not available at the time of writing.
{% endhint %}

This project follows the [Semantic Versioning principles](https://semver.org) and, contrary to upgrading a minor version (where the middle number changes) where no difficulty should be encountered, upgrading a major version (where the first number changes) is subject to significant modifications.

Version 6.0 removes everything that has been deprecated during the 5.x line. The safe path is to upgrade to the latest 5.4 release first, enable the deprecation notices, and fix them one by one following the per-version pages of this section. Once your application runs on 5.4 without a single WebAuthn deprecation, the upgrade to 6.0 is a `composer update`.

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
| Firewall | `security.firewalls.main.webauthn` | `custom_authenticator` with `WebauthnPassport` and `WebauthnBadge` |

## What is removed in 6.0.0

Everything below is removed. Each entry points to the page explaining the replacement.

### Configuration (Symfony bundle)

| Removed | Deprecated in | Migration |
|---|---|---|
| `webauthn.creation_profiles`, `webauthn.request_profiles`, `webauthn.controllers` | 5.4.0 | [From 5.3 to 5.4](from-v5.3-to-v5.4.md) |
| `webauthn.allowed_origins`, `webauthn.allow_subdomains` | 5.4.0 | [From 5.3 to 5.4](from-v5.3-to-v5.4.md) |
| `webauthn.client_override_policy` | 5.4.0 | [From 5.3 to 5.4](from-v5.3-to-v5.4.md) |
| `secured_rp_ids` (root, controllers and firewall) | 5.2.0 | [From 5.1 to 5.2](from-v5.1-to-v5.2.md) |
| `options_storage` at the controller and firewall levels | 5.2.0 | [From 5.1 to 5.2](from-v5.1-to-v5.2.md) |
| The `webauthn` firewall | 5.2.0 | [From 5.1 to 5.2](from-v5.1-to-v5.2.md) |
| `rp.name` and `rp.icon` in the creation profiles | 5.3.0 and 5.1.0 | [From 5.2 to 5.3](from-v5.2-to-v5.3.md), [From 5.0 to 5.1](from-v5.0-to-v5.1.md) |

### Classes, interfaces and methods

| Removed | Deprecated in | Replacement |
|---|---|---|
| `Webauthn\PublicKeyCredentialSource` | 5.3.0 | `Webauthn\CredentialRecord` |
| `Webauthn\Bundle\Repository\PublicKeyCredentialSourceRepositoryInterface` | 5.3.0 | `CredentialRecordRepositoryInterface` |
| `Webauthn\Bundle\Repository\CanSaveCredentialSource` and `saveCredentialSource()` | 5.3.0 | `CanSaveCredentialRecord` and `saveCredentialRecord()` |
| `Webauthn\Bundle\Repository\DoctrineCredentialSourceRepository` | 5.2.0 | Your own Doctrine repository |
| The `credentialSource` property and `getPublicKeyCredentialSource()` of the events | 5.3.0 | The `credentialRecord` property |
| `CredentialRecordConverter::toPublicKeyCredentialSource()` and `toPublicKeyCredentialSources()` | 5.3.0 | Nothing, the conversion is pointless once `PublicKeyCredentialSource` is gone |
| `PublicKeyCredentialEntity::$icon` and the `$icon` arguments | 5.1.0 | Nothing, the member left the specification |
| `PublicKeyCredentialRpEntity::$name` | 5.3.0 | Nothing, the serializer defaults the member to the Relying Party ID |
| `CollectedClientData::createFormJson()` | 5.3.0 | The Symfony Serializer |
| `PublicKeyCredentialDescriptor::AUTHENTICATOR_TRANSPORT_CABLE` | 5.3.0 | `AUTHENTICATOR_TRANSPORT_HYBRID` |
| `CeremonyStepManagerFactory::setSecuredRelyingPartyId()` and `Webauthn\CeremonyStep\CheckOrigin` | 5.2.0 | `setAllowedOrigins()` and `CheckAllowedOrigins` |
| `AuthenticatorData::getReservedForFutureUse1()` / `2()`, and the same on `WebauthnToken` | 5.4.0 | Nothing, the bits are always zero |
| `PseudoRandomFunctionInputExtensionBuilder::requiresHmacSecretMc()` | 5.4.0 | `requiresMultipleCredentialEvaluation()` |

### Composer package

`web-auth/webauthn-stimulus` is removed. The Stimulus controllers are published to npm as [`@web-auth/webauthn-stimulus`](https://www.npmjs.com/package/@web-auth/webauthn-stimulus). See [From 5.2 to 5.3](from-v5.2-to-v5.3.md).

## Breaking changes to plan for

### Sessions are invalidated

The `$reservedForFutureUse1` and `$reservedForFutureUse2` arguments are dropped from the `WebauthnToken` constructor, as well as from its `__serialize()` and `__unserialize()` methods. The serialized payload of the token therefore loses two entries and a session created by a 5.x application cannot be unserialized by a 6.0 one.

{% hint style="danger" %}
Plan to invalidate the existing sessions when you deploy 6.0.
{% endhint %}

### A secret is required for the fake credentials

`Webauthn\SimpleFakeCredentialGenerator` requires a non-empty secret. See [From 5.2 to 5.3](from-v5.2-to-v5.3.md) and [Fake Credentials](../symfony-bundle/advanced-behaviors/fake-credentials.md).

### PublicKeyCredentialUserEntity.name stays

`PublicKeyCredentialUserEntity::$name` is **not** removed: the property moves from the abstract parent class down to the user entity and keeps working exactly as before. Only the Relying Party name goes away.
