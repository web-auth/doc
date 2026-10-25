# Debugging

WebAuthn operations can be complex to debug due to the cryptographic operations and browser interactions involved. This page explains how to enable logging to troubleshoot issues during development or monitor your application in production.

## PSR-3 Logger Integration

The WebAuthn library supports [PSR-3 compatible loggers](https://www.php-fig.org/psr/psr-3/) for debugging and monitoring. You can use any PSR-3 logger implementation such as Monolog, Symfony Logger, or others.

### Setting Up a Logger

Several WebAuthn classes implement the `Webauthn\MetadataService\CanLogData` interface, allowing you to inject a PSR-3 logger:

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Psr\Log\LogLevel;
use Webauthn\AuthenticatorAttestationResponseValidator;
use Webauthn\AuthenticatorAssertionResponseValidator;

// Create a PSR-3 logger (using Monolog as example)
$logger = new Logger('webauthn');
$logger->pushHandler(new StreamHandler('path/to/webauthn.log', LogLevel::DEBUG));

// Inject logger into validators
$attestationValidator = AuthenticatorAttestationResponseValidator::create($creationCSM);
$attestationValidator->setLogger($logger);

$assertionValidator = AuthenticatorAssertionResponseValidator::create($requestCSM);
$assertionValidator->setLogger($logger);
```
{% endcode %}

## What Gets Logged

The WebAuthn library logs various events during authentication and registration:

### Registration (Attestation) Events

* Challenge generation and validation
* Client data validation
* Authenticator data parsing
* Public key extraction
* Attestation statement verification
* Certificate chain validation (when using attestation)
* Errors during any step

### Authentication (Assertion) Events

* Challenge validation
* Credential lookup
* Signature verification
* Counter validation
* User presence and verification checks
* Errors during any step

## Log Levels

The library uses standard PSR-3 log levels:

* **DEBUG**: Detailed information for debugging (enabled only in development)
* **INFO**: General informational messages
* **WARNING**: Warning messages for recoverable issues
* **ERROR**: Error messages for failures
* **CRITICAL**: Critical errors requiring immediate attention

## Example Log Output

Here's what typical log messages look like:

```
[2025-01-15 10:30:45] webauthn.DEBUG: Validating authenticator attestation response
[2025-01-15 10:30:45] webauthn.DEBUG: Challenge validated successfully
[2025-01-15 10:30:45] webauthn.DEBUG: Parsing authenticator data
[2025-01-15 10:30:45] webauthn.INFO: Credential registered successfully {"credentialId": "AQIDBAUG..."}
```

In case of errors:

```
[2025-01-15 10:31:12] webauthn.ERROR: Challenge mismatch {"expected": "abc123...", "received": "def456..."}
[2025-01-15 10:31:12] webauthn.CRITICAL: Signature verification failed
```

## Development vs Production

### Development Environment

Enable verbose DEBUG logging to see all operations:

{% code lineNumbers="true" %}
```php
<?php

// Development: Log everything
$logger = new Logger('webauthn');
$logger->pushHandler(
    new StreamHandler('php://stdout', LogLevel::DEBUG)
);
```
{% endcode %}

### Production Environment

In production, log only warnings and errors to avoid performance impact:

{% code lineNumbers="true" %}
```php
<?php

// Production: Log only warnings and above
$logger = new Logger('webauthn');
$logger->pushHandler(
    new StreamHandler('/var/log/webauthn.log', LogLevel::WARNING)
);

// Optional: Send critical errors to email or monitoring service
$logger->pushHandler(
    new \Monolog\Handler\NativeMailerHandler(
        'security@example.com',
        'WebAuthn Critical Error',
        'noreply@example.com',
        LogLevel::CRITICAL
    )
);
```
{% endcode %}

## Common Issues and Log Messages

### "Challenge mismatch"

**Cause**: The challenge sent to the client doesn't match the one stored on the server.

**Solution**: Ensure challenges are properly stored and retrieved from session/database.

### "Signature verification failed"

**Cause**: The authenticator's signature is invalid.

**Possible reasons**:
* Wrong public key used for verification
* Credential ID doesn't match the stored credential
* Data was tampered with
* Cloned authenticator (check counter logs)

### "Origin mismatch"

**Cause**: The origin in the client data doesn't match your server's expected origin.

**Solution**: Check your Relying Party ID configuration matches your domain.

### "User not present" or "User not verified"

**Cause**: User verification requirements not met.

**Solution**: Check your authenticator selection criteria configuration.

## Additional Debugging Tools

### Browser Developer Tools

Modern browsers provide WebAuthn debugging in developer tools:

* **Chrome/Edge**: DevTools → Application → WebAuthn
* **Firefox**: about:config → search for "webauthn"

### Network Inspector

Monitor the network requests to see:
* Options requests (registration/authentication)
* Response data being sent back
* HTTP errors or timeouts

### Test Authenticators

Use virtual authenticators for testing:

{% code lineNumbers="true" %}
```javascript
// Chrome DevTools Protocol
// Enable virtual authenticator in Chrome DevTools → Application → WebAuthn
// Add Virtual Authenticator → Configure options
```
{% endcode %}

## See Also

* [Symfony Bundle Debugging](../../symfony-bundle/advanced-behaviors/debugging.md) - Framework-specific logging
* [Authenticator Counter](authenticator-counter.md) - Detect cloned authenticators
* [Attestation Verification](attestation-and-metadata-statement.md) - Verify authenticator trust
