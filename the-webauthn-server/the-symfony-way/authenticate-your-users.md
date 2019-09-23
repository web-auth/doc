# Authenticate Your Users

To authenticate your users,, you need to create a `PublicKeyCredentialRequestOptions` object. You can create this object using the .... Similarly to the authentication registration process, there is another approach.

The bundle provides a factory and manages profiles to ease the creation of the options. The factory is available as a public service: `Webauthn\Bundle\Service\PublicKeyCredentialRequestOptionsFactory`. To use it, you must first create a least one profile in your configuration file.

{% code-tabs %}
{% code-tabs-item title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    request_profiles:
        acme: ~
```
{% endcode-tabs-item %}
{% endcode-tabs %}



