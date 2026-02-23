# Cross Origin Authentication

Please [refer to this page](../../pure-php/advanced-behaviours/cross-origin-authentication.md) to know more about the Cross Origin Authentication.

## Configuration

The configuration of the allowed domains can be done as follows.

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    allowed_origins:
        - 'https://acme.com'
        - 'https://acme.fr'
        - 'android:apk-key-hash://your-app-hash'
        - 'ios:bundle-id://your.bundle.id'
    allow_subdomains: true
```
{% endcode %}

## Allowed Origins Endpoint

When the allowed\_origins parameter is set, the path `/.well-known/webauthn` is enabled. This path returns a `JSON` object with allowed domains.
