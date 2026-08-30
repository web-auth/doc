---
description: Upgrade guide from 5.0 to 5.1
---

# From 5.0 to 5.1

{% hint style="success" %}
No backward compatibility break. Upgrading is a `composer update` away. A single member is deprecated.
{% endhint %}

## Deprecations

### PublicKeyCredentialEntity.icon

{% hint style="warning" %}
**Deprecated in v5.1.0**
{% endhint %}

The `icon` member has been removed from the WebAuthn specification, so the value is never used. The property, the `$icon` argument of the entity constructors and factories, and the `icon` node of the bundle creation profiles are deprecated and will be removed in 6.0.0.

```php
# Before (deprecated)
use Webauthn\PublicKeyCredentialRpEntity;

$rpEntity = PublicKeyCredentialRpEntity::create('My Application', 'example.com', 'https://example.com/logo.png');

# After
use Webauthn\PublicKeyCredentialRpEntity;

$rpEntity = PublicKeyCredentialRpEntity::create('My Application', 'example.com');
```

The same applies to `PublicKeyCredentialUserEntity`. Passing a value other than `null` triggers a deprecation notice.

With the Symfony bundle, remove the `icon` node from your creation profiles:

```yaml
webauthn:
    creation_profiles:
        default:
            rp:
                id: 'example.com'
                name: 'My Application'
                # icon: 'https://example.com/logo.png' <- remove this line
```

## Other changes

Nothing to do on your side, but worth knowing:

* the validation failure events carry the `Throwable` that caused the failure, which makes reporting and debugging much easier. See [Debugging](../pure-php/advanced-behaviours/debugging.md).
* the Stimulus controllers forward the error message returned by the server, when there is one, in the failure events.
* `http://localhost` is accepted during the origin check, so a development setup without TLS does not need a workaround anymore.
