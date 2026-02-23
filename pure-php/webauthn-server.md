# Webauthn Server

This page provides an overview of the components needed to build a WebAuthn server in pure PHP, without framework dependencies.

## Architecture Overview

A WebAuthn server handles two main operations:

1. **Registration (Attestation)**: Register new authenticators for users
2. **Authentication (Assertion)**: Verify users with their registered authenticators

Each operation involves two phases:
* **Options generation**: Create challenge and parameters for the client
* **Response validation**: Verify the client's cryptographic response

## Required Components

To set up a WebAuthn server, you need components from these categories:

### 1. Input Loading

[Input Loading](input-loading.md) services convert raw data from the browser into PHP objects:

* **Deserializers**: Transform JSON data into PHP objects
* **Decoders**: Decode base64url-encoded binary data
* **Parsers**: Parse CBOR-encoded authenticator data

### 2. Input Validation

[Input Validation](input-validation.md) services verify cryptographic signatures and data integrity:

* **Ceremony Step Managers**: Orchestrate validation steps
* **Response Validators**: Verify attestation and assertion responses
* **Algorithm Managers**: Handle cryptographic algorithms (ES256, RS256, etc.)

### 3. Data Repositories

Storage for credentials and users:

* **[Credential Record](../prerequisites/credential-record.md)**: Store and retrieve authenticator credentials
* **[User Entity](../prerequisites/user-entity.md)**: Manage user accounts

### 4. Relying Party Configuration

[Relying Party](../prerequisites/the-relying-party.md) configuration identifies your application:

* Application name and ID (domain)
* Optional icon
* Allowed origins and subdomains

## Implementation Flow

### Registration Flow

{% code lineNumbers="true" %}
```php
<?php

// 1. Create options for registration
$publicKeyCredentialCreationOptions = PublicKeyCredentialCreationOptions::create(
    $rpEntity,           // Your application
    $userEntity,         // The user being registered
    random_bytes(32),    // Challenge
    // ... additional options
);

// 2. Send options to browser
// Browser calls navigator.credentials.create()

// 3. Receive and load attestation response
$publicKeyCredential = $serializer->deserialize(
    $request->getContent(),
    PublicKeyCredential::class,
    'json'
);

// 4. Validate the response
$credentialSource = $attestationValidator->check(
    $publicKeyCredential->response,
    $publicKeyCredentialCreationOptions,
    'https://your-domain.com'
);

// 5. Store the credential
$credentialRepository->saveCredentialSource($credentialSource);
```
{% endcode %}

### Authentication Flow

{% code lineNumbers="true" %}
```php
<?php

// 1. Get user's registered credentials
$credentials = $credentialRepository->findAllForUserEntity($userEntity);
$allowedCredentials = array_map(
    fn($c) => $c->getPublicKeyCredentialDescriptor(),
    $credentials
);

// 2. Create options for authentication
$publicKeyCredentialRequestOptions = PublicKeyCredentialRequestOptions::create(
    random_bytes(32),    // Challenge
    allowCredentials: $allowedCredentials
);

// 3. Send options to browser
// Browser calls navigator.credentials.get()

// 4. Receive and validate assertion response
$credentialSource = $credentialRepository->findOneByCredentialId(
    $publicKeyCredential->rawId
);

$updatedCredential = $assertionValidator->check(
    $credentialSource,
    $publicKeyCredential->response,
    $publicKeyCredentialRequestOptions,
    'https://your-domain.com',
    $userEntity->id
);

// 5. Update credential (counter may have changed)
$credentialRepository->saveCredentialSource($updatedCredential);
```
{% endcode %}

## Minimal Setup Example

Here's a minimal working example with in-memory storage:

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\PublicKeyCredentialRpEntity;
use Webauthn\PublicKeyCredentialUserEntity;
use Webauthn\CeremonyStep\CeremonyStepManagerFactory;
use Webauthn\AuthenticatorAttestationResponseValidator;
use Webauthn\AuthenticatorAssertionResponseValidator;

// 1. Configure Relying Party
$rpEntity = PublicKeyCredentialRpEntity::create(
    'ACME WebAuthn Server',
    'acme.com'
);

// 2. Create validation services
$csmFactory = new CeremonyStepManagerFactory();
$creationCSM = $csmFactory->creationCeremony();
$requestCSM = $csmFactory->requestCeremony();

$attestationValidator = AuthenticatorAttestationResponseValidator::create($creationCSM);
$assertionValidator = AuthenticatorAssertionResponseValidator::create($requestCSM);

// 3. Implement repositories (use your storage backend)
$credentialRepository = new YourCredentialRepository();
$userRepository = new YourUserRepository();

// Now you can handle registration and authentication!
```
{% endcode %}

## Next Steps

Once you have these components set up, proceed to:

1. **[Register Authenticators](authenticator-registration.md)** - Implement the full registration flow
2. **[Authenticate Users](authenticate-your-users.md)** - Implement the authentication flow
3. **[Advanced Behaviours](advanced-behaviours/README.md)** - Customize security and UX settings

## Production Considerations

Before going to production:

* ✅ Use HTTPS (required by WebAuthn)
* ✅ Implement proper session management for challenges
* ✅ Store credentials securely in a database
* ✅ Enable logging for debugging and monitoring
* ✅ Configure proper CORS and CSP headers
* ✅ Test with multiple authenticator types (security keys, platform authenticators)
* ✅ Handle browser compatibility gracefully

## See Also

* [Prerequisites](../prerequisites/the-relying-party.md) - Understanding the basics
* [Symfony Bundle](../symfony-bundle/bundle-installation.md) - Framework integration
* [JavaScript Integration](../prerequisites/javascript.md) - Client-side implementation
