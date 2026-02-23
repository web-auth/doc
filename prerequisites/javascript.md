# JavaScript

WebAuthn requires JavaScript to interact with authenticators in the browser. This page explains the client-side implementation using the WebAuthn Browser API and recommended libraries.

## Browser API Overview

The WebAuthn specification provides two main JavaScript APIs:

1. **`navigator.credentials.create()`** - Register a new authenticator (attestation)
2. **`navigator.credentials.get()`** - Authenticate with a registered authenticator (assertion)

Both APIs are asynchronous and return Promises that resolve with credential objects.

## HTTPS Requirement

{% hint style="danger" %}
**HTTPS is mandatory** for WebAuthn to work. This applies to all domains, including `localhost`. The only exception is `localhost` in some browsers during development, but production environments must always use HTTPS.
{% endhint %}

Modern browsers enforce this requirement to prevent credential theft and man-in-the-middle attacks.

## Recommended Libraries

### @simplewebauthn/browser

We highly recommend using [@simplewebauthn/browser](https://simplewebauthn.dev/docs/packages/browser/). This library:

* Simplifies the WebAuthn API with easy-to-use functions
* Handles base64url encoding/decoding automatically
* Provides TypeScript type definitions
* Is fully compliant with the WebAuthn specification
* Works with any server implementation

**Installation:**

## Symfony UX Stimulus Controller

If you're using Symfony, the [Stimulus Controller](../symfony-ux/installation.md) provides seamless integration with forms and authentication workflows without writing JavaScript code.

## Registration (Attestation) Flow

### Using Native Browser API

{% code lineNumbers="true" %}
```javascript
// 1. Fetch registration options from your server
const optionsResponse = await fetch('/webauthn/register/options', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        username: 'john.doe@example.com',
        displayName: 'John Doe'
    })
});

const options = await optionsResponse.json();

// 2. Convert base64url strings to ArrayBuffers
const publicKeyCredentialCreationOptions = {
    ...options,
    challenge: base64urlDecode(options.challenge),
    user: {
        ...options.user,
        id: base64urlDecode(options.user.id)
    },
    excludeCredentials: options.excludeCredentials?.map(cred => ({
        ...cred,
        id: base64urlDecode(cred.id)
    }))
};

// 3. Call the WebAuthn API
const credential = await navigator.credentials.create({
    publicKey: publicKeyCredentialCreationOptions
});

// 4. Encode response for server
const attestationResponse = {
    id: credential.id,
    rawId: base64urlEncode(credential.rawId),
    type: credential.type,
    response: {
        clientDataJSON: base64urlEncode(credential.response.clientDataJSON),
        attestationObject: base64urlEncode(credential.response.attestationObject)
    }
};

// 5. Send to server for validation
await fetch('/webauthn/register', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(attestationResponse)
});
```
{% endcode %}

### Using @simplewebauthn/browser

{% code lineNumbers="true" %}
```javascript
import { startRegistration } from '@simplewebauthn/browser';

try {
    // 1. Fetch registration options from your server
    const optionsResponse = await fetch('/webauthn/register/options', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            username: 'john.doe@example.com',
            displayName: 'John Doe'
        })
    });

    const options = await optionsResponse.json();

    // 2. Start registration (handles encoding automatically)
    const attestationResponse = await startRegistration(options);

    // 3. Send to server for validation
    const verificationResponse = await fetch('/webauthn/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(attestationResponse)
    });

    const verificationResult = await verificationResponse.json();

    if (verificationResult.verified) {
        alert('Registration successful!');
    }
} catch (error) {
    console.error('Registration failed:', error);
    alert('Registration failed: ' + error.message);
}
```
{% endcode %}

## Authentication (Assertion) Flow

### Using Native Browser API

{% code lineNumbers="true" %}
```javascript
// 1. Fetch authentication options from server
const optionsResponse = await fetch('/webauthn/login/options', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        username: 'john.doe@example.com'
    })
});

const options = await optionsResponse.json();

// 2. Convert base64url strings to ArrayBuffers
const publicKeyCredentialRequestOptions = {
    ...options,
    challenge: base64urlDecode(options.challenge),
    allowCredentials: options.allowCredentials?.map(cred => ({
        ...cred,
        id: base64urlDecode(cred.id)
    }))
};

// 3. Call the WebAuthn API
const credential = await navigator.credentials.get({
    publicKey: publicKeyCredentialRequestOptions
});

// 4. Encode response for server
const assertionResponse = {
    id: credential.id,
    rawId: base64urlEncode(credential.rawId),
    type: credential.type,
    response: {
        clientDataJSON: base64urlEncode(credential.response.clientDataJSON),
        authenticatorData: base64urlEncode(credential.response.authenticatorData),
        signature: base64urlEncode(credential.response.signature),
        userHandle: credential.response.userHandle
            ? base64urlEncode(credential.response.userHandle)
            : null
    }
};

// 5. Send to server for validation
await fetch('/webauthn/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(assertionResponse)
});
```
{% endcode %}

### Using @simplewebauthn/browser

{% code lineNumbers="true" %}
```javascript
import { startAuthentication } from '@simplewebauthn/browser';

try {
    // 1. Fetch authentication options from server
    const optionsResponse = await fetch('/webauthn/login/options', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            username: 'john.doe@example.com'
        })
    });

    const options = await optionsResponse.json();

    // 2. Start authentication (handles encoding automatically)
    const assertionResponse = await startAuthentication(options);

    // 3. Send to server for validation
    const verificationResponse = await fetch('/webauthn/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(assertionResponse)
    });

    const verificationResult = await verificationResponse.json();

    if (verificationResult.verified) {
        window.location.href = '/dashboard';
    }
} catch (error) {
    console.error('Authentication failed:', error);
    alert('Authentication failed: ' + error.message);
}
```
{% endcode %}

## Base64URL Encoding/Decoding

If you're using the native API without a library, you need helper functions for base64url encoding:

{% code lineNumbers="true" %}
```javascript
function base64urlEncode(buffer) {
    const base64 = btoa(String.fromCharCode(...new Uint8Array(buffer)));
    return base64
        .replace(/\+/g, '-')
        .replace(/\//g, '_')
        .replace(/=/g, '');
}

function base64urlDecode(base64url) {
    const base64 = base64url
        .replace(/-/g, '+')
        .replace(/_/g, '/');
    const padding = '='.repeat((4 - base64.length % 4) % 4);
    const binary = atob(base64 + padding);
    const bytes = new Uint8Array(binary.length);
    for (let i = 0; i < binary.length; i++) {
        bytes[i] = binary.charCodeAt(i);
    }
    return bytes.buffer;
}
```
{% endcode %}

{% hint style="info" %}
The [@simplewebauthn/browser](https://simplewebauthn.dev/docs/packages/browser/) library handles all encoding/decoding automatically, which is why we recommend it.
{% endhint %}

## Error Handling

WebAuthn operations can fail for various reasons. Always wrap calls in try-catch blocks:

{% code lineNumbers="true" %}
```javascript
import { startAuthentication } from '@simplewebauthn/browser';

try {
    const response = await startAuthentication(options);
    // Success
} catch (error) {
    // Handle different error types
    if (error.name === 'NotAllowedError') {
        alert('Operation cancelled or timed out');
    } else if (error.name === 'InvalidStateError') {
        alert('Authenticator already registered');
    } else if (error.name === 'NotSupportedError') {
        alert('WebAuthn not supported in this browser');
    } else if (error.name === 'AbortError') {
        alert('Operation was aborted');
    } else {
        alert('Authentication failed: ' + error.message);
    }
}
```
{% endcode %}

## Common Error Types

| Error Name | Description | Common Cause |
|------------|-------------|--------------|
| `NotAllowedError` | User cancelled the operation or timeout | User cancelled prompt, 60-second timeout exceeded |
| `InvalidStateError` | Invalid state for operation | Authenticator already registered for this account |
| `NotSupportedError` | Operation not supported | Browser doesn't support WebAuthn or specific features |
| `SecurityError` | Security requirements not met | Not using HTTPS, invalid origin/RP ID |
| `AbortError` | Operation aborted | Explicitly cancelled by application code |

## Browser Compatibility

WebAuthn is supported in modern browsers:

* ✅ Chrome 67+
* ✅ Firefox 60+
* ✅ Safari 13+ (iOS and macOS)
* ✅ Edge 18+
* ✅ Opera 54+

Always check for WebAuthn support before attempting registration or authentication:

{% code lineNumbers="true" %}
```javascript
if (!window.PublicKeyCredential) {
    alert('WebAuthn is not supported in this browser');
    // Show fallback authentication method
    return;
}

// Optional: Check for specific features
if (window.PublicKeyCredential.isUserVerifyingPlatformAuthenticatorAvailable) {
    const available = await PublicKeyCredential.isUserVerifyingPlatformAuthenticatorAvailable();
    if (available) {
        console.log('Platform authenticator (Touch ID, Face ID, Windows Hello) available');
    }
}
```
{% endcode %}

## Browser Autofill / Conditional UI

Modern browsers support autofill for WebAuthn credentials, allowing users to select passkeys from password autofill:

{% code lineNumbers="true" %}
```javascript
// Check if conditional UI is available
const conditionalUIAvailable = await PublicKeyCredential.isConditionalMediationAvailable();

if (conditionalUIAvailable) {
    // Add 'conditional' mediation to allow autofill
    const credential = await navigator.credentials.get({
        publicKey: options,
        mediation: 'conditional'  // Enable autofill UI
    });
}
```
{% endcode %}

For Symfony UX users, enable this with the `useBrowserAutofill` option in the Stimulus controller configuration.

## Development Setup

### Local HTTPS

For development with HTTPS on localhost:

**Option 1: Use mkcert (Recommended)**

```bash
# Install mkcert
brew install mkcert  # macOS
# or apt install mkcert  # Linux

# Create local CA
mkcert -install

# Generate certificate for localhost
mkcert localhost 127.0.0.1 ::1
```

**Option 2: Symfony CLI (for Symfony projects)**

```bash
symfony server:ca:install
symfony serve
```

The Symfony CLI automatically sets up HTTPS for local development.

## See Also

* [@simplewebauthn/browser Documentation](https://simplewebauthn.dev/docs/packages/browser/)
* [Symfony UX Integration](../symfony-ux/installation.md) - Framework integration
* [Pure PHP Server Setup](../pure-php/webauthn-server.md) - Server-side implementation
* [MDN WebAuthn API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API) - Browser API reference
