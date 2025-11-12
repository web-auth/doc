---
description: Authenticator details and how to manage them
---

# Credential Source

## Credential Source Class

After the registration of an authenticator, you will get a `Webauthn\PublicKeyCredentialSource` object. This object contains all the credential data needed to perform user authentication:

* **publicKeyCredentialId**: The unique identifier of the credential (binary string)
* **type**: The credential type (always `"public-key"`)
* **transports**: Supported transports (`usb`, `nfc`, `ble`, `internal`, `hybrid`)
* **attestationType**: The attestation type used during registration (`none`, `basic`, `self`, `attca`, `anonca`)
* **trustPath**: The trust path containing certificate chain information
* **aaguid**: The Authenticator AAGUID (Authenticator Attestation GUID)
* **credentialPublicKey**: The public key in COSE format (binary string)
* **userHandle**: The user identifier (binary string)
* **counter**: The signature counter to detect cloned authenticators
* **backupEligible**: Whether the credential can be backed up (since 5.1)
* **backupStatus**: Whether the credential is currently backed up (since 5.1)
* **uvInitialized**: Whether user verification was initialized (since 5.1)
* **otherUI**: Optional additional UI hints (array)

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\PublicKeyCredentialSource;

// After successful registration, you receive a PublicKeyCredentialSource object
$credentialSource = $authenticatorAttestationResponseValidator->check(
    $authenticatorAttestationResponse,
    $publicKeyCredentialCreationOptions,
    'https://example.com'
);

// Access credential properties
$credentialId = $credentialSource->publicKeyCredentialId;
$userId = $credentialSource->userHandle;
$counter = $credentialSource->counter;
$transports = $credentialSource->transports; // ['usb', 'nfc']

// Get the descriptor for authentication
$descriptor = $credentialSource->getPublicKeyCredentialDescriptor();
```
{% endcode %}

## Credential Source Repository

Since 4.6.0 and except if you use the Symfony bundle, there is no interface to implement or abstract class to extend, making it easy to integrate into your application.

Your repository needs to provide two main operations:

1. **Save a credential** after registration or counter update
2. **Find credentials** by credential ID or user handle

{% hint style="success" %}
Whatever database you use (MySQL, PostgreSQL, MongoDB…), it is not necessary to create foreign key relationships between your users and the Credential Sources. The `userHandle` property is sufficient to link credentials to users.
{% endhint %}

### Repository Example

Here's a simple example using an array storage (for demonstration purposes):

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

namespace App\Repository;

use Webauthn\PublicKeyCredentialSource;

final class InMemoryCredentialSourceRepository
{
    private array $credentials = [];

    public function saveCredentialSource(PublicKeyCredentialSource $publicKeyCredentialSource): void
    {
        $this->credentials[$publicKeyCredentialSource->publicKeyCredentialId] = $publicKeyCredentialSource;
    }

    public function findOneByCredentialId(string $publicKeyCredentialId): ?PublicKeyCredentialSource
    {
        return $this->credentials[$publicKeyCredentialId] ?? null;
    }

    public function findAllForUserEntity(string $userHandle): array
    {
        return array_filter(
            $this->credentials,
            static fn(PublicKeyCredentialSource $credential): bool =>
                $credential->userHandle === $userHandle
        );
    }
}
```
{% endcode %}

{% hint style="info" %}
For production use, implement your repository with your preferred storage backend (Doctrine ORM, PDO, MongoDB, Redis, etc.). See the [Symfony Bundle section](../symfony-bundle/entities-with-doctrine.md) for a complete Doctrine example.
{% endhint %}

### Important Notes

* **Store credentials securely**: Credentials contain sensitive cryptographic material
* **Index by credential ID**: Lookups during authentication require fast credential ID queries
* **Index by user handle**: Registration and credential listing require fast user handle queries
* **Counter updates**: Update the counter after each successful authentication to detect cloned authenticators
