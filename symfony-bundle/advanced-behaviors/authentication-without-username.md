# Authentication without username

With Webauthn, it is possible to authenticate a user without username. This behavior implies several constraints:

1. During the registration of the authenticator, a [Resident Key must have been asked](../../deep-into-the-framework/authenticator-selection-criteria.md#resident-key),
2. The user verification is required,
3. The list of allowed authenticators must be empty

{% hint style="info" %}
In case of failure, you should continue with the standard authentication process i.e. by asking the username of the user.
{% endhint %}

The bundle configuration should have a profile with the constraints listed above:

```javascript
webauthn:
    credential_repository: '…'
    user_repository: '…'
    creation_profiles:
        default:
            rp:
                name: 'My application'
                id: 'example.com'
            authenticator_selection_criteria:
                require_resident_key: true
                user_verification: !php/const Webauthn\AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_REQUIRED
    request_profiles:
        default:
            rp_id: 'example.com'
            user_verification: !php/const Webauthn\AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_REQUIRED
```
