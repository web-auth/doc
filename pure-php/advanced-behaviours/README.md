# Advanced Behaviours

This section covers advanced WebAuthn features and customization options for Pure PHP implementations.

## Overview

While the basic WebAuthn implementation covers most use cases, you may need to customize certain behaviors to meet specific security or user experience requirements. These advanced topics help you fine-tune your WebAuthn integration.

## Available Topics

### Security & Validation

* **[Debugging](debugging.md)** - Enable debug logging to troubleshoot WebAuthn operations
* **[Authenticator Counter](authenticator-counter.md)** - Detect cloned authenticators using signature counters
* **[Attestation and Metadata Statement](attestation-and-metadata-statement.md)** - Verify authenticator attestations and trust anchors

### User Experience

* **[User Verification](user-verification.md)** - Configure biometric or PIN requirements
* **[Authenticator Selection Criteria](authenticator-selection-criteria.md)** - Control which authenticators can be used
* **[Authentication without Username](authentication-without-username.md)** - Enable resident keys for passwordless login

### Credential Management

* **[Signal API](signal-api.md)** - Inform clients about credential status changes (new in 5.3.0)
* **[Backup Events](backup-events.md)** - React to backup eligibility and status changes (new in 5.3.0)
* **[Conditional Create](conditional-create.md)** - Auto-register credentials after password login (new in 5.3.0)

### Technical Configuration

* **[Authenticator Algorithms](authenticator-algorithms.md)** - Configure supported cryptographic algorithms
* **[Extensions](extensions.md)** - Use WebAuthn extensions for additional features
* **[Cross Origin Authentication](cross-origin-authentication.md)** - Handle localhost and development environments

## When to Use Advanced Features

Most applications will work fine with the default WebAuthn configuration. Consider these advanced features when:

* **Security is paramount**: Use attestation verification and counter checking for high-security applications
* **User experience matters**: Enable usernameless authentication and optimize authenticator selection
* **Debugging issues**: Enable logging when troubleshooting registration or authentication problems
* **Compliance requirements**: Some regulations may require specific attestation or verification settings

## Quick Start

For basic WebAuthn implementation, start with these pages:

1. [Webauthn Server](../webauthn-server.md) - Set up the basic server components
2. [Authenticator Registration](../authenticator-registration.md) - Register user authenticators
3. [Authenticate Your Users](../authenticate-your-users.md) - Perform authentication

Then return to this section for advanced customization.

{% hint style="info" %}
If you're using the Symfony Bundle, see the [Symfony Bundle Advanced Behaviors](../../symfony-bundle/advanced-behaviors/README.md) section for framework-specific implementations.
{% endhint %}
