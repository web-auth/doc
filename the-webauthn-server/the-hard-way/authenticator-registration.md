# Register Authenticators

During this step, your application will send a challenge to the device. The device will resolve this challenge by adding information and digitally signing the data.

The application will check the response from the device and get its credential ID. This ID will be used for further authentication requests.

## Creation Request

To associate a device to a user, you need to instantiate a `Webauthn\PublicKeyCredentialCreationOptions` object.

It will need:

* [The Relying Party](../../pre-requisites/the-relaying-party.md)
* [The User data](../../pre-requisites/user-entity-repository.md)
* A challenge \(random binary string\)
* A list of supported public key parameters i.e. an algorithm list \(at least one\)
* A timeout \(optional\)
* A list of public key credential to exclude from the registration process \(optional\)
* [The Authenticator Selection Criteria](../../deep-into-the-framework/authenticator-selection-criteria.md) \(e.g. user presence requirement\)
* [Attestation conveyance preference](../../deep-into-the-framework/attestation-and-metadata-statement.md) \(optional\)
* [Extensions](../../deep-into-the-framework/extensions.md) \(optional\)

Let see an example of `PublicKeyCredentialCreationOptions` object. The following example is a possible Public Key Creation page for a dummy user "@cypher-Angel-3000".

```php
<?php

declare(strict_types=1);

use Cose\Algorithms;
use Webauthn\PublicKeyCredentialDescriptor;
use Webauthn\PublicKeyCredentialCreationOptions;
use Webauthn\PublicKeyCredentialParameters;
use Webauthn\PublicKeyCredentialRpEntity;
use Webauthn\PublicKeyCredentialUserEntity;

// RP Entity
$rpEntity = new PublicKeyCredentialRpEntity(
    'My Super Secured Application', //Name
    'foo.example.com',              //ID
    null                            //Icon
);

// User Entity
$userEntity = new PublicKeyCredentialUserEntity(
    '@cypher-Angel-3000',                   //Name
    '123e4567-e89b-12d3-a456-426655440000', //ID
    'Mighty Mike',                          //Display name
    null                                    //Icon
);

// Challenge
$challenge = random_bytes(16);

// Public Key Credential Parameters
$publicKeyCredentialParametersList = [
    new PublicKeyCredentialParameters('public-key', Algorithms::COSE_ALGORITHM_ES256),
    new PublicKeyCredentialParameters('public-key', Algorithms::COSE_ALGORITHM_RS256),
];

// Devices to exclude
$excludedPublicKeyDescriptors = [
    new PublicKeyCredentialDescriptor(PublicKeyCredentialDescriptor::CREDENTIAL_TYPE_PUBLIC_KEY, 'ABCDEFGH…'),
];

$publicKeyCredentialCreationOptions = new PublicKeyCredentialCreationOptions(
    $rpEntity,
    $userEntity,
    $challenge,
    $publicKeyCredentialParametersList
);
```

The options object can be converted into JSON and sent to the authenticator [using a JS script](../../pre-requisites/javascript.md).

{% hint style="warning" %}
It is important to store the user entity and the options object \(e.g. in the session\) for the next step; they will be needed to check the response from the device.
{% endhint %}

## Creation Response

What you receive must be a JSON object that looks like as follow:

```javascript
{
    "id":"KVb8CnwDjpgAo[…]op61BTLaa0tczXvz4JrQ23usxVHA8QJZi3L9GZLsAtkcVvWObA",
    "type":"public-key",
    "rawId":"KVb8CnwDjpgAo[…]rQ23usxVHA8QJZi3L9GZLsAtkcVvWObA==",
    "response":{
        "clientDataJSON":"eyJjaGFsbGVuZ2UiOiJQbk1hVjBVTS[…]1iUkdHLUc4Y3BDSdGUifQ==",
        "attestationObject":"o2NmbXRmcGFja2VkZ2F0dFN0bXSj[…]YcGhf"
    }
}
```

There are two steps to perform with this object:

* Load the data
* Verify it with the creation options set above

### Data Loading

Now that all components are set, we can load the data we receive using the _Public Key Credential Loader_ service \(variable `$publicKeyCredential`\).

```php
<?php

declare(strict_types=1);

$data = '
{
    "id":"KVb8CnwDjpgAo[…]op61BTLaa0tczXvz4JrQ23usxVHA8QJZi3L9GZLsAtkcVvWObA",
    "type":"public-key",
    "rawId":"KVb8CnwDjpgAo[…]rQ23usxVHA8QJZi3L9GZLsAtkcVvWObA==",
    "response":{
        "clientDataJSON":"eyJjaGFsbGVuZ2UiOiJQbk1hVjBVTS[…]1iUkdHLUc4Y3BDSdGUifQ==",
        "attestationObject":"o2NmbXRmcGFja2VkZ2F0dFN0bXSj[…]YcGhf"
    }
}';

$publicKeyCredential = $publicKeyCredentialLoader->load($data);
```

If no exception is thrown, you can go to the next step: the verification.

### Response Verification

Now we have a fully loaded Public Key Credential object, but we need now to make sure that:

1. The authenticator response is of type `AuthenticatorAttestationResponse`
2. This response is valid.

The first is easy to perform:

```php
<?php

declare(strict_types=1);

use Webauthn\AuthenticatorAttestationResponse;

$authenticatorAttestationResponse = $publicKeyCredential->getResponse();
if (!$authenticatorAttestationResponse instanceof AuthenticatorAttestationResponse) {
    //e.g. process here with a redirection to the public key creation page. 
}
```

The second step is the verification against

* The Public Key Creation Options we created earlier,
* The HTTP request

The Authenticator Attestation Response Validator service \(variable `$authenticatorAttestationResponseValidator`\) will check everything for you: challenge, origin, attestation statement and much more.

{% hint style="info" %}
The library needs PSR-7 requests. In the example below, we use `nyholm/psr7-server` to get that request.
{% endhint %}

```php
<?php

declare(strict_types=1);

use Nyholm\Psr7\Factory\Psr17Factory;
use Nyholm\Psr7Server\ServerRequestCreator;

$psr17Factory = new Psr17Factory();
$creator = new ServerRequestCreator(
    $psr17Factory, // ServerRequestFactory
    $psr17Factory, // UriFactory
    $psr17Factory, // UploadedFileFactory
    $psr17Factory  // StreamFactory
);

$serverRequest = $creator->fromGlobals();

$publicKeyCredentialSource = $authenticatorAttestationResponseValidator->check(
    $authenticatorAttestationResponse,
    $publicKeyCredentialCreationOptions,
    $serverRequest
);
```

If no exception is thrown, the response is valid. You can store the Public Key Credential Source `($publicKeyCredentialSource`\) and associate it to the user entity.

{% hint style="info" %}
The way you store and associate these objects to the user is out of scope of this library. However, please note that these objects implement `\JsonSerializable` and have a static method `createFromJson(string $json)`. This will allow you to serialize the objects into JSON and easily go back an object.
{% endhint %}

{% hint style="warning" %}
If you have just registered a new user, don’t forget to store it in your database as well.
{% endhint %}

