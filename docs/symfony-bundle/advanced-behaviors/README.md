# Advanced behaviors

This section covers advanced WebAuthn features and customization options for Symfony Bundle implementations.

## Overview

The Symfony WebAuthn Bundle provides sensible defaults for most applications. However, you can customize various behaviors to meet specific security requirements or enhance user experience.

## Available topics

### Security features

* **[Fake Credentials](fake-credentials.md)** - Prevent user enumeration attacks with fake credentials
* **[Authenticator Counter](authenticator-counter.md)** - Detect cloned authenticators
* **[Attestation and Metadata Statement](attestation-and-metadata-statement.md)** - Verify authenticator trust
* **[Debugging](debugging.md)** - Enable debug logging for troubleshooting

### User experience

* **[User Verification](user-verification.md)** - Configure biometric or PIN requirements
* **[Authenticator Selection Criteria](authenticator-selection-criteria.md)** - Control authenticator types
* **[Authentication without Username](authentication-without-username.md)** - Passwordless authentication with resident keys
* **[Register Additional Authenticators](register-authenticators.md)** - Allow users to add backup authenticators

### Policy & discovery

* **[Client Override Policy](client-override-policy.md)** - Control which options clients can override (new in 5.3.0)
* **[Passkey Endpoints](passkey-endpoints.md)** - Expose .well-known/passkey-endpoints discovery (new in 5.3.0)

### Technical configuration

* **[Extensions](extensions.md)** - Use WebAuthn protocol extensions
* **[Cross Origin Authentication](cross-origin-authentication.md)** - Development environment configuration

## Configuration vs code

The Symfony Bundle allows configuration through:

1. **YAML Configuration** - Most settings can be configured in `config/packages/webauthn.yaml`
2. **Custom Services** - Advanced behaviors require creating custom service classes
3. **Event Listeners** - Hook into the authentication process with Symfony events

## Symfony-specific features

The bundle provides several Symfony-specific features not available in pure PHP:

* **Firewall Integration** - Seamless integration with Symfony Security
* **Dependency Injection** - All services available through the service container
* **Configuration Profiles** - Multiple authentication profiles for different use cases
* **Event System** - React to WebAuthn events throughout your application

## Quick configuration example

Here's a common advanced configuration:

```yaml title="config/packages/webauthn.yaml" linenums="1"
webauthn:
    credential_repository: 'App\Repository\WebauthnCredentialRepository'
    user_repository: 'App\Repository\UserRepository'

    # Enable debugging in development
    logger: 'monolog.logger'

    # Custom counter checker to detect cloned authenticators
    counter_checker: 'App\Security\CustomCounterChecker'

    creation_profiles:
        default:
            rp:
                name: 'My Application'
                id: 'example.com'

            # Require resident keys for passwordless auth
            authenticator_selection_criteria:
                authenticator_attachment: !php/const Webauthn\AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_PLATFORM
                require_resident_key: true
                user_verification: !php/const Webauthn\AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_REQUIRED

    request_profiles:
        default:
            rp_id: 'example.com'
            user_verification: !php/const Webauthn\AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED
```

## See also

* [Firewall Configuration](../firewall.md) - Basic Symfony Security setup
* [Configuration References](../configuration-references.md) - Complete configuration options
* [Pure PHP Advanced Behaviours](../../pure-php/advanced-behaviours/README.md) - Framework-agnostic implementations

!!! info

    Start with the basic bundle setup in [Bundle Installation](../bundle-installation.md) before diving into advanced behaviors.
