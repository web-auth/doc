# Passkey Endpoints

{% hint style="info" %}
**New in v5.3.0**
{% endhint %}

The [W3C Passkey Endpoints specification](https://w3c.github.io/webappsec-passkey-endpoints/) defines a well-known URL (`/.well-known/passkey-endpoints`) that allows passkey providers to discover where users can manage their passkeys on your application.

This enables password managers and platform authenticators to link directly to your passkey enrollment and management pages.

## Configuration

Enable the passkey endpoints in your Symfony bundle configuration:

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    passkey_endpoints:
        enabled: true
        enroll: 'https://example.com/passkeys/register'
        manage: 'https://example.com/passkeys/manage'
        prf_usage_details: 'https://example.com/passkeys/prf-info'
```
{% endcode %}

All URL fields are optional. You can enable only the endpoints relevant to your application:

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    passkey_endpoints:
        enabled: true
        enroll: 'https://example.com/register'
        manage: 'https://example.com/settings/passkeys'
```
{% endcode %}

## Configuration Options

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `enabled` | boolean | Yes | Enable the `.well-known/passkey-endpoints` route |
| `enroll` | string | No | URL to the passkey enrollment/creation interface |
| `manage` | string | No | URL to the passkey management interface |
| `prf_usage_details` | string | No | URL to informational page about PRF extension usage |

## Response Format

When enabled, a GET request to `/.well-known/passkey-endpoints` returns a JSON response:

```json
{
    "enroll": "https://example.com/passkeys/register",
    "manage": "https://example.com/passkeys/manage",
    "prfUsageDetails": "https://example.com/passkeys/prf-info"
}
```

{% hint style="warning" %}
According to the specification, all URLs must be absolute HTTPS URLs.
{% endhint %}

## How It Works

When the passkey endpoints are enabled, the bundle automatically:

1. Registers a `PasskeyEndpointsController` as a service
2. Adds the `/.well-known/passkey-endpoints` route to the router
3. Returns the configured URLs as a JSON response

No additional controller or route configuration is needed.

## See Also

* [W3C Passkey Endpoints Specification](https://w3c.github.io/webappsec-passkey-endpoints/) - Official specification
* [Configuration References](../configuration-references.md) - Full bundle configuration
