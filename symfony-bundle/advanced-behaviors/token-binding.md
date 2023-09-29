# Token Binding

To change the token binding behaviour, you can change the dedicated parameter in the bundle configuration.

{% code title="config/packages/webauthn.yaml" %}
```yaml
webauthn:
    token_binding_support_handler: Webauthn\TokenBinding\TokenBindingNotSupportedHandler
```
{% endcode %}
