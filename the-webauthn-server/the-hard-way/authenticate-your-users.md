# Authenticate Your Users

During this step, your application will send a challenge to the list of registered devices of the user. The security token will resolve this challenge by adding information and digitally signing the data.

## Assertion Request

To perform a user authentication using a security device, you need to instantiate a `Webauthn\PublicKeyCredentialRequestOptions` object.

Let’s say you want to authenticate the user we used earlier. This options object will need:

* A challenge \(random binary string\)
* A timeout \(optional\)
* [The Relying Party ID](../../pre-requisites/the-relaying-party.md) i.e. your application domain \(optional\)
* The list with the allowed credentials
* [The user verification requirement](../../deep-into-the-framework/user-verification.md) \(optional\)
* [Extensions](../../deep-into-the-framework/extensions.md) \(optional\)

The `PublicKeyCredentialRequestOptions` object is designed to be easily serialized into a JSON object. This will ease the integration into an HTML page or through an API endpoint.

### Allowed Credentials

The user trying to authenticate must have registered at least one device. For this user, you have to get all `Webauthn\PublicKeyCredentialDescriptor` associated to his account.

```php
// We gather all registered authenticators for this user
$registeredAuthenticators = $publicKeyCredentialSourceRepository->findAllForUserEntity($userEntity);

// We don’t need the Credential Sources, just the associated Descriptors
$allowedCredentials = array_map(
    static function (PublicKeyCredentialSource $credential): PublicKeyCredentialDescriptor {
        return $credential->getPublicKeyCredentialDescriptor();
    },
    $registeredAuthenticators
);
```

### User Verification

Eligible authenticators are filtered and only capable of satisfying this requirement will interact with the user. Please refer to the [User Verification page](../../deep-into-the-framework/user-verification.md) for all possible values.

### Extensions

Please refer to the [Extension page](../../deep-into-the-framework/extensions.md) to know how to manage authentication extensions.

### Example

```php
<?php

declare(strict_types=1);

use Webauthn\PublicKeyCredentialRequestOptions;

// List of registered PublicKeyCredentialDescriptor classes associated to the user
$registeredAuthenticators = $publicKeyCredentialSourceRepository->findAllForUserEntity($userEntity);
$allowedCredentials = array_map(
    static function (PublicKeyCredentialSource $credential): PublicKeyCredentialDescriptor {
        return $credential->getPublicKeyCredentialDescriptor();
    },
    $registeredAuthenticators
);

// Public Key Credential Request Options
$publicKeyCredentialRequestOptions = new PublicKeyCredentialRequestOptions(
    random_bytes(32),                                                           // Challenge
    60000,                                                                      // Timeout
    'foo.example.com',                                                          // Relying Party ID
    $allowedCredentials                                                         // Extensions
);
```

## Response Handling

The way you receive this response is out of scope of this library. In the previous example, the data is part of the query string, but it can be done through a POST request body or a request header.

What you receive must be a JSON object that looks like as follows:

```javascript
{
    "id":"KVb8CnwDjpgAo[…]op61BTLaa0tczXvz4JrQ23usxVHA8QJZi3L9GZLsAtkcVvWObA",
    "type":"public-key",
    "rawId":"KVb8CnwDjpgAo[…]rQ23usxVHA8QJZi3L9GZLsAtkcVvWObA==",
    "response":{
        "clientDataJSON":"eyJjaGFsbGVuZ2UiOiJQbk1hVjBVTS[…]1iUkdHLUc4Y3BDSdGUifQ==",
        "authenticatorData":"Y0EWbxTqi9hWTO[…]4aust69iUIzlwBfwABDw==",
        "signature":"MEQCIHpmdruQLs[…]5uwbtlPNOFM2oTusx2eg==",
        "userHandle":""
    }
}
```

There are two steps to perform with this object:

* Load the data
* Verify the loaded data against the assertion options set above

### Data Loading

This step is exactly the same as the one described in [Public Key Credential Creation](authenticator-registration.md) process.

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

### Response Verification

Now we have a fully loaded Public Key Credential object, but we need now to make sure that:

1. The authenticator response is of type `AuthenticatorAssertionResponse`
2. This response is valid.

The first is easy to perform:

```php
<?php

declare(strict_types=1);

use Webauthn\AuthenticatorAssertionResponse;

$authenticatorAssertionResponse = $publicKeyCredential->getResponse();
if (!$authenticatorAssertionResponse instanceof AuthenticatorAssertionResponse) {
    //e.g. process here with a redirection to the public key login/MFA page. 
}
```

The second step is the verification against the Public Key Assertion Options we created earlier.

The Authenticator Assertion Response Validator service \(variable `$authenticatorAssertionResponseValidator`\) will check everything for you.

```php
<?php

declare(strict_types=1);

use Symfony\Component\HttpFoundation\Request;

$request = Request::createFromGlobals();

$publicKeyCredentialSource = $authenticatorAssertionResponse->check(
    $publicKeyCredential->getRawId(),
    $authenticatorAssertionResponse,
    $publicKeyCredentialRequestOptions,
    $request,
    $userHandle
);
```

If no exception is thrown, the response is valid and you can continue the authentication of the user.

{% hint style="info" %}
The Public Key Credential Source returned allows you to know which device was used by the user.
{% endhint %}

