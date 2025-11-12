# Extensions

{% hint style="info" %}
This page is about an advanced feature. You can skip it if you are new with WebAuthn.
{% endhint %}

WebAuthn extensions are optional features that extend the core WebAuthn specification to support additional use cases and capabilities. They allow authenticators and browsers to provide extra functionality beyond basic authentication.

## What Are Extensions?

Extensions provide a standardized way to:

* Request additional capabilities from authenticators
* Retrieve extra information during registration or authentication
* Enable specific features like credential protection or large blob storage
* Maintain backward compatibility with legacy systems (like U2F)

## How Extensions Work

Extensions work in both directions:

1. **Extension Inputs** - Sent from the Relying Party to the authenticator via the browser
2. **Extension Outputs** - Returned from the authenticator to the Relying Party after processing

{% hint style="warning" %}
**Important**: Authenticators may ignore extension inputs if they don't support them. Extension outputs are **not guaranteed** and your application should handle their absence gracefully.
{% endhint %}

## Common WebAuthn Extensions

### Standard Extensions

The most commonly used WebAuthn extensions include:

| Extension | Purpose | Use Case |
|-----------|---------|----------|
| `appid` | U2F backward compatibility | Allow existing U2F credentials to work with WebAuthn |
| `credProps` | Credential properties | Determine if a credential is discoverable (resident key) |
| `largeBlob` | Large blob storage | Store additional data (up to ~2KB) with credentials |
| `credProtect` | Credential protection | Require user verification for certain credentials |
| `minPINLength` | Minimum PIN length | Enforce PIN complexity requirements |
| `hmacSecret` | HMAC secret | Generate symmetric secrets for encryption |

### Registry

All standard extensions are listed in the **IANA WebAuthn Extensions Registry**:

[https://www.iana.org/assignments/webauthn/webauthn.xhtml](https://www.iana.org/assignments/webauthn/webauthn.xhtml)

## Library Support

This library provides:

* ✅ **Core extension infrastructure** - Ready to handle extension inputs and outputs
* ✅ **AppID extension** - Built-in support for U2F backward compatibility
* ⚠️ **Custom extensions** - You need to implement extension handlers for other extensions

{% hint style="warning" %}
Most extensions require custom implementation. It is up to you to create the extension handlers depending on which extensions you want to support.
{% endhint %}

## Basic Usage Example

Here's how to add an extension input when creating credential options:

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\AuthenticationExtensions\AuthenticationExtension;
use Webauthn\AuthenticationExtensions\AuthenticationExtensions;
use Webauthn\PublicKeyCredentialCreationOptions;

// Create extension inputs
$extensions = AuthenticationExtensions::create([
    AuthenticationExtension::create('credProps', true)  // Request credential properties
]);

// Add extensions to creation options
$publicKeyCredentialCreationOptions = PublicKeyCredentialCreationOptions::create(
    rp: $rpEntity,
    user: $userEntity,
    challenge: random_bytes(32),
    extensions: $extensions
);
```
{% endcode %}

After registration, check the extension outputs:

{% code lineNumbers="true" %}
```php
// After successful registration
$extensionOutputs = $publicKeyCredential->response->getAuthenticatorData()->extensions;

// Check if credential is discoverable (resident key)
if (isset($extensionOutputs['credProps']['rk'])) {
    $isDiscoverable = $extensionOutputs['credProps']['rk'];
    echo $isDiscoverable ? 'Resident key created' : 'Non-resident key';
}
```
{% endcode %}

## Practical Use Cases

### 1. U2F Migration (appid Extension)

Allow existing U2F credentials to work with your WebAuthn implementation:

{% code lineNumbers="true" %}
```php
$extensions = AuthenticationExtensions::create([
    AuthenticationExtension::create('appid', 'https://example.com')
]);

$publicKeyCredentialRequestOptions = PublicKeyCredentialRequestOptions::create(
    random_bytes(32),
    extensions: $extensions
);
```
{% endcode %}

### 2. Credential Properties (credProps Extension)

Determine if a credential supports resident keys (usernameless authentication):

{% code lineNumbers="true" %}
```php
$extensions = AuthenticationExtensions::create([
    AuthenticationExtension::create('credProps', true)
]);

// After registration, check:
$rk = $extensionOutputs['credProps']['rk'] ?? false;
```
{% endcode %}

### 3. Large Blob Storage (largeBlob Extension)

Store additional encrypted data with the credential:

{% code lineNumbers="true" %}
```php
// Write blob during registration
$extensions = AuthenticationExtensions::create([
    AuthenticationExtension::create('largeBlob', [
        'support' => 'required',
        'write' => base64_encode('encrypted user data')
    ])
]);
```
{% endcode %}

## Implementation Considerations

### Check Extension Support

Not all authenticators support all extensions. Always:

1. Check extension outputs to see what was actually processed
2. Have fallback behavior when extensions aren't supported
3. Don't rely on extensions for critical security features

### Custom Extension Handlers

For extensions beyond `appid`, you'll need to implement:

1. **Extension Output Checker** - Validates extension outputs from authenticators
2. **Extension Input Builder** - Constructs proper extension input objects

See the [Pure PHP Extensions Guide](../pure-php/advanced-behaviours/extensions.md) or [Symfony Bundle Extensions Guide](../symfony-bundle/advanced-behaviors/extensions.md) for detailed implementation instructions.

## When to Use Extensions

Use extensions when you need:

* ✅ **U2F compatibility** - Migrate existing U2F credentials
* ✅ **Credential metadata** - Know if credentials are resident keys
* ✅ **Additional storage** - Store encrypted data with credentials
* ✅ **Enhanced protection** - Enforce user verification policies

Avoid extensions if:

* ❌ Basic registration/authentication is sufficient
* ❌ You need universal compatibility (not all devices support extensions)
* ❌ You're just getting started with WebAuthn

## See Also

* [Pure PHP Extensions Implementation](../pure-php/advanced-behaviours/extensions.md) - Detailed extension implementation
* [Symfony Extensions Configuration](../symfony-bundle/advanced-behaviors/extensions.md) - Framework-specific setup
* [IANA WebAuthn Extensions Registry](https://www.iana.org/assignments/webauthn/webauthn.xhtml) - Official extension list
* [WebAuthn Specification](https://www.w3.org/TR/webauthn-2/#sctn-extensions) - Extension mechanism details
