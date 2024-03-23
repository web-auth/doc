# Register Authenticators

During this step, your application will send a challenge to the device. The device will resolve this challenge by adding information and digitally signing the data.

The application will check the response from the device and get its credential ID. This ID will be used for further authentication requests.

## Creation Request

To associate a device to a user, you need to instantiate a `Webauthn\PublicKeyCredentialCreationOptions` object.

It will need:

* The Relying Party
* The User data
* A challenge (random binary string)
* A list of supported public key parameters i.e. an algorithm list (at least one)

Optionally, you can customize the following parameters:

* A timeout
* A list of public key credential to exclude from the registration process
* The Authenticator Selection Criteria
* Attestation conveyance preference
* Extensions

Let’s see an example of the `PublicKeyCredentialCreationOptions` object. The following example is a possible Public Key Creation page for a dummy user "@cypher-Angel-3000".

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\PublicKeyCredentialCreationOptions;
use Webauthn\PublicKeyCredentialRpEntity;
use Webauthn\PublicKeyCredentialUserEntity;

// RP Entity i.e. the application
$rpEntity = PublicKeyCredentialRpEntity::create(
    'My Super Secured Application', //Name
    'foo.example.com',              //ID
    null                            //Icon
);

// User Entity
$userEntity = PublicKeyCredentialUserEntity::create(
    '@cypher-Angel-3000',                   //Name
    '123e4567-e89b-12d3-a456-426655440000', //ID
    'Mighty Mike',                          //Display name
    null                                    //Icon
);

// Challenge
$challenge = random_bytes(16);

$publicKeyCredentialCreationOptions =
    PublicKeyCredentialCreationOptions::create(
        $rpEntity,
        $userEntity,
        $challenge
    )
;
```
{% endcode %}

The options object can be converted into JSON and sent to the authenticator [using the API](https://developer.mozilla.org/en-US/docs/Web/API/Web\_Authentication\_API).

```php
$jsonObject = json_encode($publicKeyCredentialCreationOptions);
```

{% hint style="warning" %}
It is important to store the user entity and the options object (e.g. in the session) for the next step. The data will be needed to check the response from the device.
{% endhint %}

You can change the default values for each and all options

{% code lineNumbers="true" %}
```php
$publicKeyCredentialCreationOptions =
    PublicKeyCredentialCreationOptions::create(
        $rpEntity,
        $userEntity,
        $challenge,
        pubKeyCredParams: $publicKeyCredentialParametersList,
        authenticatorSelection: AuthenticatorSelectionCriteria::create(),
        attestation: PublicKeyCredentialCreationOptions::ATTESTATION_CONVEYANCE_PREFERENCE_NONE,
        excludeCredentials: $excludedPublicKeyDescriptors,
        timeout: 30_000,
    )
;
```
{% endcode %}

### Public Key Credential Parameters

The argument `pubKeyCredParams` contains a list of `Webauthn\PublicKeyCredentialParameters` objects that refer to COSE algorithms. The authenticators must use one of the algorithms in this list, respecting the order of preference on this list.

An empty list corresponds to the default algorithms that are `ES256` and `RS256` (in this order). Those two algorithms are required by the specification.

{% code lineNumbers="true" %}
```php
use Cose\Algorithms;
use Webauthn\PublicKeyCredentialParameters;

$publicKeyCredentialParametersList = [
    PublicKeyCredentialParameters::create('public-key', Algorithms::COSE_ALGORITHM_ES256K), // More interesting algorithm
    PublicKeyCredentialParameters::create('public-key', Algorithms::COSE_ALGORITHM_ES256),  //      ||
    PublicKeyCredentialParameters::create('public-key', Algorithms::COSE_ALGORITHM_RS256),  //      || 
    PublicKeyCredentialParameters::create('public-key', Algorithms::COSE_ALGORITHM_PS256),  //      \/
    PublicKeyCredentialParameters::create('public-key', Algorithms::COSE_ALGORITHM_ED256),  // Less interesting algorithm
];
```
{% endcode %}

{% hint style="warning" %}
Customizing this list may lead to unexpected behavior. Please use with caution.
{% endhint %}

### Authenticator Selection

Please read detail [on this page](advanced-behaviours/authenticator-selection-criteria.md).

### Attestation

Please read detail [on this page](broken-reference).

### Exclude Credentials

When the user already registered authenticators, you can pass a list of `Webauthn\PublicKeyCredentialDescriptor` objects as argument to avoid registering multiple times the same authenticator.

## Creation Response

What you receive must be a JSON object that looks like as follow:

{% code lineNumbers="true" %}
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
{% endcode %}

There are two steps to perform with this object:

* [Load the data](input-loading.md)
* [Verify it with the creation options set above](input-validation.md)

### Data Loading

Now that all components are set, we can load the data we receive using the [_Serializer_](webauthn-server.md#the-serializer) (variable `$serializer`).

<pre class="language-php" data-line-numbers><code class="lang-php"><strong>&#x3C;?php
</strong>
declare(strict_types=1);

use Webauthn\PublicKeyCredential;

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

$publicKeyCredential = $serializer->deserialize(
    $data,
    PublicKeyCredential::class,
    'json'
);
</code></pre>

If no exception is thrown, you can go to the next step: the verification.

### Response Verification

Now we have a fully loaded Public Key Credential object, but we need now to make sure that:

1. The authenticator response is of type `AuthenticatorAttestationResponse`
2. This response is valid.

The first step is easy to perform:

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\AuthenticatorAttestationResponse;

if (!$publicKeyCredential->response instanceof AuthenticatorAttestationResponse) {
    //e.g. process here with a redirection to the public key creation page. 
}
```
{% endcode %}

The second step is the verification against

* The Public Key Creation Options we created earlier,
* The URI host

The Authenticator Attestation Response Validator service (variable `$authenticatorAttestationResponseValidator`) will check everything for you: challenge, origin, attestation statement and much more.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

$publicKeyCredentialSource = $authenticatorAttestationResponseValidator->check(
    $authenticatorAttestationResponse,
    $publicKeyCredentialCreationOptions,
    'my-application.com'
);
```
{% endcode %}

If no exception is thrown, the response is valid. You can store the Public Key Credential Source (`$publicKeyCredentialSource`).

{% hint style="info" %}
The way you store and associate these objects to the user is out of scope of this library. Please note that these objects implement `\JsonSerializable` and have a static method `createFromJson(string $json)`. This will allow you to serialize the objects into JSON and easily go back to an object.
{% endhint %}
