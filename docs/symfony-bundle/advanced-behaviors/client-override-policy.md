# Client override policy

!!! info

    **New in v5.3.0**

The Client Override Policy provides granular control over which WebAuthn options clients can override via request parameters. This allows you to define strict server-side defaults while optionally allowing clients to customize specific fields within constrained boundaries.

!!! warning

    **5.4 helper-side API recommended.** When you build options through the [Options Helpers](../options-helpers.md), attach the policy with `withClientOverrides(...)` and use the typed `ClientOverridePolicy::fromRules()` factory documented under *Client overrides* on that page. The YAML `creation_profiles[].client_override_policy` form below stays supported but is deprecated alongside `creation_profiles` in 5.4.

## Overview

When building WebAuthn options from a profile, the server uses configured defaults. With the Client Override Policy, you can control whether HTTP request parameters can override these defaults and which values are acceptable.

Each policy field has:
* **enabled**: Whether the client can override this field at all
* **allowed_values**: An optional list of accepted values (if omitted, all valid values are allowed)

## Helper-side, typed API (5.4)

`ClientOverridePolicy::fromRules()` and `ClientOverrideRule` give you a named-argument, typed alternative to the YAML / nested-array form. Both shapes are first-class — pick whichever fits your use case best.

```php
use Webauthn\Bundle\Policy\ClientOverridePolicy;
use Webauthn\Bundle\Policy\ClientOverrideRule;

$policy = ClientOverridePolicy::fromRules(
    userVerification:        ClientOverrideRule::restrictTo(['preferred', 'required']),
    authenticatorAttachment: ClientOverrideRule::restrictTo(['platform', 'cross-platform']),
    residentKey:             ClientOverrideRule::any(),
    extensions:              ClientOverrideRule::any(),
);

return $this->options
    ->forCreation('example.com', $this->guesser)
    ->withClientOverrides($policy)
    ->build($request);
```

* Pass `null` (the default) for fields the client must NOT be able to override.
* `ClientOverrideRule::any()` accepts any value the client submits.
* `ClientOverrideRule::restrictTo($allowedValues)` restricts to a list.

## Configuration

The example below shows every field with an explicit value. You only need to declare the fields you want to change — the rest fall back to the [defaults](#configurable-fields) listed in the next section.

```yaml title="config/packages/webauthn.yaml" linenums="1"
webauthn:
    creation_profiles:
        default:
            rp:
                id: 'example.com'
            client_override_policy:
                user_verification:
                    enabled: true
                    allowed_values: ['required', 'preferred']
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
                mediation:
                    enabled: false   # disabled by default; opt-in
                    allowed_values: ['default', 'conditional']
```

## Configurable fields

| Field | Default | Default Allowed Values | Description |
|-------|---------|------------------------|-------------|
| `user_verification` | **disabled** | `required`, `preferred` | Whether the client can request a specific user verification level. Disabled by default to prevent downgrade attacks; `discouraged` is excluded from the defaults even when you enable the override. |
| `authenticator_attachment` | enabled | `platform`, `cross-platform` | Whether the client can request a specific authenticator type |
| `resident_key` | enabled | `required`, `preferred`, `discouraged` | Whether the client can request resident key behavior |
| `attestation_conveyance` | enabled | `none`, `indirect`, `direct`, `enterprise` | Whether the client can request attestation preference |
| `extensions` | enabled | N/A | Whether the client can provide additional extensions |
| `mediation` | **disabled** | `default`, `conditional` | Whether the client can request the [Conditional Create](../../pure-php/advanced-behaviours/conditional-create.md) flow (auto-register) — opt-in. |

!!! warning

    **Why `user_verification` is disabled by default.** Allowing a client to override the user verification requirement makes it possible to request `discouraged` against a profile that mandates `required`, which silently weakens the security guarantee. If you enable this override, keep `discouraged` out of `allowed_values` and **always re-check the `UV` flag at the application layer** — see [User Verification](../../webauthn-in-a-nutshell/user-verification.md#checking-the-uv-flag).

## Examples

### Restrict to platform authenticators only

Force platform authenticators (like Touch ID, Windows Hello) and prevent clients from requesting cross-platform authenticators:

```yaml title="config/packages/webauthn.yaml" linenums="1"
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

### Lock down all options

For high-security scenarios, disable all client overrides:

```yaml title="config/packages/webauthn.yaml" linenums="1"
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

### Allow only specific verification levels

Allow the client to choose between `required` and `preferred`, but not `discouraged`:

```yaml title="config/packages/webauthn.yaml" linenums="1"
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

### Opt-in conditional create from the client

Conditional mediation is the only override that is **disabled by default**. Enable it on a per-profile basis if you want the JavaScript client to request the [Conditional Create](../../pure-php/advanced-behaviours/conditional-create.md) flow without the profile being permanently configured for it:

```yaml title="config/packages/webauthn.yaml" linenums="1"
webauthn:
    creation_profiles:
        default:
            rp:
                id: 'example.com'
            client_override_policy:
                mediation:
                    enabled: true
                    allowed_values: ['default', 'conditional']
```

## How it works

The `ClientOverridePolicy` class determines the effective value for each field:

1. If no request value is provided, the profile default is used
2. If the override is disabled for that field, the profile default is used
3. If the request value is not in the `allowed_values` list, the profile default is used
4. Otherwise, the request value is used

This ensures that the server always maintains control over the final options, while allowing controlled flexibility for client-side customization.

## See also

* [Configuration References](../configuration-references.md) - Full bundle configuration
* [Authenticator Selection Criteria](authenticator-selection-criteria.md) - Selection criteria details
* [User Verification](user-verification.md) - User verification options
