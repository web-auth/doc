# User Verification

The easiest way to manage the user verification requirements in your Symfony application is by using the creation and request profiles.

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    …
    creation_profiles:
        default:
            rp:
                name: 'My Application'
                id: 'example.com'
            authenticator_selection_criteria:
                user_verification: !php/const Webauthn\AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED
    request_profiles:
        default:
            rp_id: 'example.com'
            user_verification: !php/const Webauthn\AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED
```
{% endcode %}
