# Authenticate Your Users

During this step, your application will send a challenge to the list of registered devices of the user. The security token will resolve this challenge by adding information and digitally signing the data.

## Assertion Request

To perform a user authentication using a security device, you need to instantiate a `Webauthn\PublicKeyCredentialRequestOptions` object.

Let’s say you want to authenticate the user we used earlier. This options object will need:

* A challenge (random binary string)
* The list with the allowed credentials (may be an option in certain circumstances)

Optionally, you can customize the following parameters:

* A timeout
* The Relying Party ID i.e. your application domain
* The user verification requirement
* Extensions

The `PublicKeyCredentialRequestOptions` object is designed to be easily serialized into a JSON object. This will ease the integration into an HTML page or through an API endpoint.

{% hint style="info" %}
The timeout default value is set to `null`. If you want to set a value, pleaase read the following recommended behavior showed in the specification:

* If the user verification is `discouraged`, timeout should be between 30 and 180 seconds
* If the user verification is `preferred` or `required`, the range is 300 to 600 seconds (5 to 10 minutes)
{% endhint %}

### Allowed Credentials

The user trying to authenticate must have registered at least one device. For this user, you have to get all `Webauthn\PublicKeyCredentialDescriptor` associated to his account.

```php
use Webauthn\PublicKeyCredentialSource;

// We gather all registered authenticators for this user
// $publicKeyCredentialSourceRepository corresponds to your own service
// The purpose of the fictive method findAllForUserEntity is to return all credential source objects
// registered by the user.
$registeredAuthenticators = $publicKeyCredentialSourceRepository->findAllForUserEntity($userEntity);

// We don’t need the Credential Sources, just the associated Descriptors
$allowedCredentials = array_map(
    static function (PublicKeyCredentialSource $credential): PublicKeyCredentialDescriptor {
        return $credential->getPublicKeyCredentialDescriptor();
    },
    $registeredAuthenticators
);
```

{% hint style="info" %}
For usernameless authentication, please read the [dedicated page](advanced-behaviours/authentication-without-username.md). In this case no Public Key Credential Descriptors should be passed to the the options.
{% endhint %}

### Example

```php
<?php

declare(strict_types=1);

use Webauthn\PublicKeyCredentialDescriptor;
use Webauthn\PublicKeyCredentialRequestOptions;
use Webauthn\PublicKeyCredentialSource;

// List of registered PublicKeyCredentialDescriptor classes associated to the user
$registeredAuthenticators = $publicKeyCredentialSourceRepository->findAllForUserEntity($userEntity);
$allowedCredentials = array_map(
    static function (PublicKeyCredentialSource $credential): PublicKeyCredentialDescriptor {
        return $credential->getPublicKeyCredentialDescriptor();
    },
    $registeredAuthenticators
);

// Public Key Credential Request Options
$publicKeyCredentialRequestOptions =
    PublicKeyCredentialRequestOptions::create(
        random_bytes(32), // Challenge
        allowCredentials: $allowedCredentials
    )
;
```

### User Verification

Eligible authenticators are filtered and only capable of satisfying this requirement will interact with the user. Please refer to the [User Verification page](advanced-behaviours/user-verification.md) for all possible values.

```php
<?php

declare(strict_types=1);

use Webauthn\PublicKeyCredentialDescriptor;
use Webauthn\PublicKeyCredentialRequestOptions;
use Webauthn\PublicKeyCredentialSource;

// Public Key Credential Request Options
$publicKeyCredentialRequestOptions =
    PublicKeyCredentialRequestOptions::create(
        random_bytes(32), // Challenge
        userVerification: PublicKeyCredentialRequestOptions::USER_VERIFICATION_REQUIREMENT_REQUIRED
    )
;
```

### Extensions

Please refer to the [Extension page](../webauthn-in-a-nutshell/extensions.md) to know how to manage authentication extensions.

```php
<?php

declare(strict_types=1);

use Webauthn\PublicKeyCredentialRequestOptions;
use Webauthn\AuthenticationExtensions\AuthenticationExtensionsClientInputs;
use Webauthn\AuthenticationExtensions\AuthenticationExtension;

// Public Key Credential Request Options
$publicKeyCredentialRequestOptions =
    PublicKeyCredentialRequestOptions::create(
        random_bytes(32), // Challenge
        extensions: AuthenticationExtensionsClientInputs::create([
            AuthenticationExtension::create('loc', true),
            AuthenticationExtension::create('txAuthSimple', 'Please log in with a registered authenticator'),
        ])
    )
;
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

$publicKeyCredential = $serializer->deserialize($data, PublicKeyCredential::class, 'json');
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

if (!$publicKeyCredential->response instanceof AuthenticatorAssertionResponse) {
    //e.g. process here with a redirection to the public key login/MFA page. 
}
```

The second step is the verification against the Public Key Assertion Options we created earlier.

The Authenticator Assertion Response Validator service (variable `$authenticatorAssertionResponseValidator`) will check everything for you.

```php
<?php

declare(strict_types=1);

$publicKeyCredentialSource =  $publicKeyCredentialSourceRepository->findOneByCredentialId(
    $publicKeyCredential->rawId
);
if ($publicKeyCredentialSource === null) {
   // Throw an exception if the credential is not found.
   // It can also be rejected depending on your security policy (e.g. disabled by the user because of loss)
}

$publicKeyCredentialSource = $authenticatorAssertionResponseValidator->check(
    $publicKeyCredentialSource,
    $authenticatorAssertionResponse,
    $publicKeyCredentialRequestOptions,
    'my-application.com'
);

// Optional, but highly recommended, you can save the credential source as it may be modified
// during the verification process (counter may be higher).
$publicKeyCredentialSourceRepository->saveCredential($publicKeyCredentialSource);

```

If no exception is thrown, the response is valid and you can continue the authentication of the user.
