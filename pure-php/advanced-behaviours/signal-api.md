# Signal API

{% hint style="info" %}
**New in v5.3.0**
{% endhint %}

The WebAuthn Signal API allows relying parties to send signals to the client (browser/platform) about credential status. This helps clients maintain accurate credential metadata and improves the user experience by keeping passkey lists up to date.

## What Is the Signal API?

When users manage their passkeys on the server side (removing credentials, updating profile information), the client platform may still display outdated information. The Signal API provides a standardized way to inform the client about:

* Which credentials are still valid for a user
* Updated user details (name, display name)
* Credentials that are no longer recognized by the server

## Signal Types

### AllAcceptedCredentials

Informs the client about all credentials that the server currently recognizes for a given user. This allows the client to remove any credentials that are no longer valid.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\PublicKeyCredentialDescriptor;
use Webauthn\PublicKeyCredentialRpEntity;
use Webauthn\PublicKeyCredentialUserEntity;
use Webauthn\Signal\AllAcceptedCredentials;

$rpEntity = PublicKeyCredentialRpEntity::create(id: 'example.com');

$userEntity = PublicKeyCredentialUserEntity::create(
    'john.doe',
    $userHandle,
    'John Doe'
);

// List of credential descriptors still valid for this user
$acceptedCredentials = [
    PublicKeyCredentialDescriptor::create('public-key', $credentialId1),
    PublicKeyCredentialDescriptor::create('public-key', $credentialId2),
];

$signal = new AllAcceptedCredentials($rpEntity, $userEntity, $acceptedCredentials);
```
{% endcode %}

### CurrentUserDetails

Informs the client about updated user details. Use this when a user changes their username or display name to keep the client's passkey list accurate.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\PublicKeyCredentialRpEntity;
use Webauthn\PublicKeyCredentialUserEntity;
use Webauthn\Signal\CurrentUserDetails;

$rpEntity = PublicKeyCredentialRpEntity::create(id: 'example.com');

$userEntity = PublicKeyCredentialUserEntity::create(
    'new.username',        // Updated username
    $userHandle,
    'New Display Name'     // Updated display name
);

$signal = new CurrentUserDetails($rpEntity, $userEntity);
```
{% endcode %}

### UnknownCredential

Informs the client that a specific credential is not recognized by the server. This can occur when a credential has been deleted server-side or was never registered.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\PublicKeyCredentialDescriptor;
use Webauthn\PublicKeyCredentialRpEntity;
use Webauthn\Signal\UnknownCredential;

$rpEntity = PublicKeyCredentialRpEntity::create(id: 'example.com');

$unknownCredential = PublicKeyCredentialDescriptor::create('public-key', $credentialId);

$signal = new UnknownCredential($rpEntity, $unknownCredential);
```
{% endcode %}

## Serialization

Signals can be serialized to JSON using the Symfony Serializer. The framework provides dedicated denormalizers for each signal type.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Symfony\Component\Serializer\Normalizer\AbstractObjectNormalizer;

// Serialize the signal to JSON
$json = $serializer->serialize(
    $signal,
    'json',
    [AbstractObjectNormalizer::SKIP_NULL_VALUES => true]
);

// The JSON output follows the W3C Signal API format
// For AllAcceptedCredentials:
// {
//     "rpId": "example.com",
//     "userId": "...",
//     "allAcceptedCredentialIds": ["...", "..."]
// }
```
{% endcode %}

## Use Cases

### After Credential Deletion

When a user removes a passkey from your application, send an `AllAcceptedCredentials` signal with the remaining credentials so the client can update its list.

### After Profile Update

When a user changes their username or display name, send a `CurrentUserDetails` signal so the client displays the correct information in its passkey picker.

### During Authentication

If an authentication attempt references a credential that doesn't exist in your database, send an `UnknownCredential` signal to help the client clean up orphaned passkeys.

## See Also

* [WebAuthn Signal API Specification](https://github.com/nicovil/webauthn-signal-api) - W3C specification
* [Credential Record](../../prerequisites/credential-record.md) - Credential storage
