# Client Override Policy

{% hint style="info" %}
**New in v5.3.0**
{% endhint %}

The Client Override Policy provides granular control over which WebAuthn options clients can override via request parameters. This allows you to define strict server-side defaults while optionally allowing clients to customize specific fields within constrained boundaries.

## Overview

When building WebAuthn options from a profile, the server uses configured defaults. With the Client Override Policy, you can control whether HTTP request parameters can override these defaults and which values are acceptable.

Each policy field has:
* **enabled**: Whether the client can override this field at all
* **allowed_values**: An optional list of accepted values (if omitted, all valid values are allowed)

## Configuration

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    creation_profiles:
        default:
            rp:
                id: 'example.com'
            client_override_policy:
                user_verification:
                    enabled: true
                    allowed_values: ['required', 'preferred', 'discouraged']
                authenticator_attachment:
                    enabled: true
                    allowed_values: ['platform', 'cross-platform']
                resident_key:
                    enabled: true
                    allowed_values: ['required', 'preferred', 'discouraged']
                attestation_conveyance:
                    enabled: true
                    allowed_values: ['none', 'indirect', 'direct', 'enterprise']
                extensions:
                    enabled: true
```
{% endcode %}

## Configurable Fields

| Field | Default | Allowed Values | Description |
|-------|---------|----------------|-------------|
| `user_verification` | enabled | `required`, `preferred`, `discouraged` | Whether the client can request a specific user verification level |
| `authenticator_attachment` | enabled | `platform`, `cross-platform` | Whether the client can request a specific authenticator type |
| `resident_key` | enabled | `required`, `preferred`, `discouraged` | Whether the client can request resident key behavior |
| `attestation_conveyance` | enabled | `none`, `indirect`, `direct`, `enterprise` | Whether the client can request attestation preference |
| `extensions` | enabled | N/A | Whether the client can provide additional extensions |

## Examples

### Restrict to Platform Authenticators Only

Force platform authenticators (like Touch ID, Windows Hello) and prevent clients from requesting cross-platform authenticators:

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    creation_profiles:
        passkeys_only:
            rp:
                id: 'example.com'
            authenticator_selection_criteria:
                authenticator_attachment: platform
            client_override_policy:
                authenticator_attachment:
                    enabled: false  # Client cannot change this
```
{% endcode %}

### Lock Down All Options

For high-security scenarios, disable all client overrides:

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    creation_profiles:
        high_security:
            rp:
                id: 'example.com'
            authenticator_selection_criteria:
                user_verification: required
                require_resident_key: true
            client_override_policy:
                user_verification:
                    enabled: false
                authenticator_attachment:
                    enabled: false
                resident_key:
                    enabled: false
                attestation_conveyance:
                    enabled: false
                extensions:
                    enabled: false
```
{% endcode %}

### Allow Only Specific Verification Levels

Allow the client to choose between `required` and `preferred`, but not `discouraged`:

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    creation_profiles:
        default:
            rp:
                id: 'example.com'
            client_override_policy:
                user_verification:
                    enabled: true
                    allowed_values: ['required', 'preferred']
```
{% endcode %}

## How It Works

The `ClientOverridePolicy` class determines the effective value for each field:

1. If no request value is provided, the profile default is used
2. If the override is disabled for that field, the profile default is used
3. If the request value is not in the `allowed_values` list, the profile default is used
4. Otherwise, the request value is used

This ensures that the server always maintains control over the final options, while allowing controlled flexibility for client-side customization.

## See Also

* [Configuration References](../configuration-references.md) - Full bundle configuration
* [Authenticator Selection Criteria](authenticator-selection-criteria.md) - Selection criteria details
* [User Verification](user-verification.md) - User verification options
