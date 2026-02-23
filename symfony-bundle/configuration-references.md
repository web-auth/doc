# Configuration References

## Configuration

With Flex, you have a minimal configuration file installed through a Flex Recipe. You must set the repositories you have just created. You also have to modify the environment variables `RELYING_PARTY_ID` and `RELYING_PARTY_NAME`.

You may also need to adjust other parameters.

If you don't use Flex, hereafter an example of configuration file:

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
#    logger: null # PSR-3 compatible logging service
    credential_repository: 'Webauthn\Bundle\Repository\DummyPublicKeyCredentialSourceRepository' # CREATE YOUR REPOSITORY AND CHANGE THIS!
    user_repository: 'Webauthn\Bundle\Repository\DummyPublicKeyCredentialUserEntityRepository' # CREATE YOUR REPOSITORY AND CHANGE THIS!
#    allowed_origins: # List of allowed origins for WebAuthn operations (new in 5.2.0)
#        - 'https://example.com'
#        - 'https://app.example.com'
#        - 'android:apk-key-hash://your-app-hash' # For Android FIDO2
#        - 'ios:bundle-id://your.bundle.id' # For iOS
#    allow_subdomains: false # Allow subdomains when validating origins (new in 5.2.0)
    creation_profiles: # Authenticator registration profiles
        default: # Unique name of the profile
            rp: # Relying Party information
                name: '%env(RELYING_PARTY_NAME)%' # CHANGE THIS! or create the corresponding env variable
                id: '%env(RELYING_PARTY_ID)%' # Please adapt the env file with the correct relying party ID or set null
#                icon: null # Secured image (data:// scheme)
#            challenge_length: 32
#            timeout: 60000
#            hide_existing_credentials: false # Hide existing credentials during registration (new in 5.3.0)
#            conditional_create: false # Enable Conditional Create for auto-registration (new in 5.3.0)
#            authenticator_selection_criteria:
#                authenticator_attachment: !php/const Webauthn\AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_NO_PREFERENCE
#                require_resident_key: false
#                user_verification: !php/const Webauthn\AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED
#            hints: [] # WebAuthn hints: security-key, client-device, hybrid (new in 5.3.0)
#            extensions:
#                loc: true
#            public_key_credential_parameters: # You should not change this list
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_EdDSA #Order is important. Preferred algorithms go first
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_ES256
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_ES256K
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_ES384
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_ES512
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_RS256
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_RS384
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_RS512
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_PS256
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_PS384
#                - !php/const Cose\Algorithms::COSE_ALGORITHM_PS512
#            attestation_conveyance: !php/const Webauthn\PublicKeyCredentialCreationOptions::ATTESTATION_CONVEYANCE_PREFERENCE_NONE
#            client_override_policy: # Granular control over client overrides (new in 5.3.0)
#                user_verification:
#                    enabled: true
#                    allowed_values: ['required', 'preferred', 'discouraged']
#                authenticator_attachment:
#                    enabled: true
#                    allowed_values: ['platform', 'cross-platform']
#                resident_key:
#                    enabled: true
#                    allowed_values: ['required', 'preferred', 'discouraged']
#                attestation_conveyance:
#                    enabled: true
#                    allowed_values: ['none', 'indirect', 'direct', 'enterprise']
#                extensions:
#                    enabled: true
    request_profiles: # Authentication profiles
        default: # Unique name of the profile
            rp_id: '%env(RELYING_PARTY_ID)%' # Please adapt the env file with the correct relying party ID or set null
#            challenge_length: 32
#            timeout: 60000
#            user_verification: !php/const Webauthn\AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED
#            hints: [] # WebAuthn hints: security-key, client-device, hybrid (new in 5.3.0)
#            extensions:
#                loc: true
#    passkey_endpoints: # .well-known/passkey-endpoints discovery (new in 5.3.0)
#        enabled: false
#        enroll: 'https://example.com/passkeys/register'
#        manage: 'https://example.com/passkeys/manage'
#        prf_usage_details: 'https://example.com/passkeys/prf-info'
#    metadata:
#        enabled: false
#        mds_repository: 'App\Repository\MetadataStatementRepository'
#        status_report_repository: 'App\Repository\StatusReportRepository'
#        certificate_chain_checker: 'App\Security\CertificateChainChecker'
```
{% endcode %}

### Creation Profiles

{% hint style="success" %}
If you don't create the `creation_profiles` section, a `default` profile is set.
{% endhint %}

#### Relying Party (rp)

The Relying Party corresponds to your application. Please refer [to this page](../prerequisites/the-relying-party.md) for more information.

{% hint style="warning" %}
The parameter `id` is optional but highly recommended.
{% endhint %}

#### Challenge Length

By default, the length of the challenge is 32 bytes. You may need to select a smaller or higher length. This length can be configured for each profile:

{% code title="app/config/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            rp:
                name: 'ACME WebAuthn Server'
            challenge_length: 16
```
{% endcode %}

#### Timeout

The default timeout is set to 60 seconds (60 000 milliseconds). You can change this value as follows:

{% code title="app/config/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            rp:
                name: 'ACME WebAuthn Server'
            timeout: 30000
```
{% endcode %}

{% hint style="info" %}
For v4.0+, the timeout is set to `null`. The values recommended by the specification are as follows:

* If the user verification is `discouraged`, timeout should be between 30 and 180 seconds
* If the user verification is `preferred` or `required`, the range is 300 to 600 seconds (5 to 10 minutes)

These behaviors are not necessarily followed by the web browsers.
{% endhint %}

#### Authenticator Selection Criteria

This set of options allows you to select authenticators depending on their capabilities. The values are described in [the advanced concepts](../pure-php/advanced-behaviours/authenticator-selection-criteria.md) of the protocol.

{% code title="app/config/webauthn.yaml" lineNumbers="true" fullWidth="false" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            rp:
                name: 'ACME WebAuthn Server'
            authenticator_selection_criteria:
                authenticator_attachment: !php/const Webauthn\AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_PLATFORM
                require_resident_key: true
                user_verification: !php/const Webauthn\AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_REQUIRED
```
{% endcode %}

#### Public Key Credential Parameters

This option indicates the algorithms allowed for your application. By default, a large list of algorithms is defined, but you can add custom algorithms or reduce the list.

{% hint style="info" %}
The order is important. Preferred algorithms go first.
{% endhint %}

{% hint style="warning" %}
It is not recommended to change the default list unless you exactly know what you are doing.
{% endhint %}

{% code title="app/config/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            rp:
                name: 'ACME WebAuthn Server'
            public_key_credential_parameters:
                - !php/const Cose\Algorithms::COSE_ALGORITHM_ES256
                - !php/const Cose\Algorithms::COSE_ALGORITHM_RS256
```
{% endcode %}

#### Attestation Conveyance

If you need the [attestation of the authenticator](../webauthn-in-a-nutshell/attestation-and-metadata-statement.md), you can specify the preference regarding attestation conveyance during credential generation.

{% hint style="warning" %}
The use of Attestation Statements is generally not recommended unless you REALLY need this information.
{% endhint %}

{% code title="app/config/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            rp:
                name: 'ACME WebAuthn Server'
            attestation_conveyance: !php/const Webauthn\PublicKeyCredentialCreationOptions::ATTESTATION_CONVEYANCE_PREFERENCE_DIRECT
```
{% endcode %}

#### Hints

{% hint style="info" %}
**New in v5.3.0:** Support for WebAuthn hints to guide user authentication choices.
{% endhint %}

Hints provide guidance to the user agent about how the user might authenticate. This can help improve the user experience by suggesting the most appropriate authentication method.

{% code title="app/config/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            rp:
                name: 'ACME WebAuthn Server'
            hints:
                - 'security-key'
                - 'client-device'
                - 'hybrid'
    request_profiles:
        acme:
            rp_id: 'example.com'
            hints:
                - 'security-key'
                - 'client-device'
```
{% endcode %}

Available hint values:
- `security-key`: External security key (like YubiKey)
- `client-device`: Platform authenticator (like Touch ID, Windows Hello)
- `hybrid`: Hybrid transport (like phone as authenticator)

#### Hide Existing Credentials

{% hint style="info" %}
**New in v5.3.0:** Exposed in bundle configuration.
{% endhint %}

When set to `true`, the server will automatically add the user's existing credentials to the `excludeCredentials` list in the creation options. This prevents the user from re-registering the same authenticator.

{% code title="app/config/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            rp:
                name: 'ACME WebAuthn Server'
            hide_existing_credentials: true
```
{% endcode %}

#### Conditional Create

{% hint style="info" %}
**New in v5.3.0**
{% endhint %}

Enable Conditional Create (auto-register) for a profile. When `true`, user presence can be `false` during registration, allowing silent credential creation after the user has authenticated via another method (e.g., password).

{% code title="app/config/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    creation_profiles:
        passkey_upgrade:
            rp:
                id: 'example.com'
            conditional_create: true
            authenticator_selection_criteria:
                require_resident_key: true
```
{% endcode %}

See [Conditional Create](../pure-php/advanced-behaviours/conditional-create.md) for detailed usage.

#### Client Override Policy

{% hint style="info" %}
**New in v5.3.0**
{% endhint %}

Control which WebAuthn options can be overridden by client request parameters. See [Client Override Policy](advanced-behaviors/client-override-policy.md) for detailed configuration.

{% code title="app/config/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            rp:
                id: 'example.com'
            client_override_policy:
                user_verification:
                    enabled: true
                    allowed_values: ['required', 'preferred']
                authenticator_attachment:
                    enabled: false  # Prevent client override
```
{% endcode %}

#### Extensions

You can set as many extensions as you want in the profile. Please also [refer to this page](../webauthn-in-a-nutshell/extensions.md) for more information.

{% hint style="info" %}
The example below is totally fictive. Some extensions are [defined in the specification](https://www.w3.org/TR/webauthn/#sctn-defined-extensions) but the support depends on the authenticators, on the browsers and on the relying parties (your applications).
{% endhint %}

{% code title="app/config/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    creation_profiles:
        acme:
            rp:
                name: 'ACME WebAuthn Server'
            extensions:
                loc: true
                txAuthSimple: 'Please add your new authenticator'
```
{% endcode %}

### Request Profiles

{% hint style="success" %}
If you don't create the `request_profiles` section, a `default` profile is set.
{% endhint %}

The parameters for the request profiles (i.e. the authentication) are very similar to the creation profiles. The only difference is that you don't need all the detail of the Relying Party, but only its ID (i.e. its domain).

{% code title="app/config/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    request_profiles:
        acme:
            rp_id: 'example.com'
```
{% endcode %}

Please note that all parameters are optional. The following configuration is perfectly valid. However, and as mentioned above, the parameter `id` is highly recommended.

{% code title="app/config/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    request_profiles:
        acme: ~
```
{% endcode %}

### Passkey Endpoints

{% hint style="info" %}
**New in v5.3.0**
{% endhint %}

Enable the `.well-known/passkey-endpoints` discovery endpoint. See [Passkey Endpoints](advanced-behaviors/passkey-endpoints.md) for detailed configuration.

{% code title="app/config/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    passkey_endpoints:
        enabled: true
        enroll: 'https://example.com/passkeys/register'
        manage: 'https://example.com/passkeys/manage'
```
{% endcode %}
