# User Authentication

### Credention Request Options

To authenticate you user, you need to send a `Webauthn\PublicKeyCredentialRequestOptions` object. using your `$server` object, call the method `generatePublicKeyCredentialRequestOptions` 

In general, to authenticate your user you will ask them for their username first. With this username and [your user repository](../../pre-requisites/user-entity-repository.md), you will find the associated `Webauthn\PublicKeyCredentialUserEntity`.

And with the user entity you will get all associated Public Key Credential Source objects. The credential list is used to build the Public Key Credential Request Options.

```php
<?php

use Webauthn\PublicKeyCredentialRequestOptions;
use Webauthn\PublicKeyCredentialSource;
use Webauthn\PublicKeyCredentialUserEntity;

// UseEntity found using the username.
$userEntity = $userEntityRepository->findWebauthnUserByUsername('john.doe');

/** Optional: this avoid multiple registration of the same authenticator with the user account **/
// Get the list of authenticators associated to the user
$credentialSources = $credentialSourceRepository->findAllForUserEntity($userEntity);

// Convert the Credential Sources into Public Key Credential Descriptors
$allowedCredentials = array_map(function (PublicKeyCredentialSource $credential) {
return $credential->getPublicKeyCredentialDescriptor();
}, $credentialSources);
/** End of optional part**/

// We generate the set of options.
$ublicKeyCredentialRequestOptions = $server->generatePublicKeyCredentialRequestOptions(
    PublicKeyCredentialRequestOptions::USER_VERIFICATION_REQUIREMENT_DISCOURAGED, // Optional
    $allowedCredentials                                                           // Optional
);
```

Now send the options to the authenticator using your favorite Javascript framework, library or the example availbale in [the Javascript page](../../pre-requisites/javascript.md).

### Response Verification

When the authenticator send you the computed response \(i.e. the user touched the button, fingerprint reader, submitted the PIN…\), you can load it and check it.

The authenticator response looks similar to the following example:

```javascript
{
    "id":"LFdoCFJTyB82ZzSJUHc-c72yraRc_1mPvGX8ToE8su39xX26Jcqd31LUkKOS36FIAWgWl6itMKqmDvruha6ywA",
    "rawId":"LFdoCFJTyB82ZzSJUHc-c72yraRc_1mPvGX8ToE8su39xX26Jcqd31LUkKOS36FIAWgWl6itMKqmDvruha6ywA",
    "response":{
        "authenticatorData":"SZYN5YgOjGh0NBcPZHZgW4_krrmihjLHmVzzuoMdl2MBAAAAAA",
        "signature":"MEYCIQCv7EqsBRtf2E4o_BjzZfBwNpP8fLjd5y6TUOLWt5l9DQIhANiYig9newAJZYTzG1i5lwP-YQk9uXFnnDaHnr2yCKXL",
        "userHandle":"",
        "clientDataJSON":"eyJjaGFsbGVuZ2UiOiJ4ZGowQ0JmWDY5MnFzQVRweTBrTmM4NTMzSmR2ZExVcHFZUDh3RFRYX1pFIiwiY2xpZW50RXh0ZW5zaW9ucyI6e30sImhhc2hBbGdvcml0aG0iOiJTSEEtMjU2Iiwib3JpZ2luIjoiaHR0cDovL2xvY2FsaG9zdDozMDAwIiwidHlwZSI6IndlYmF1dGhuLmdldCJ9"
    },
    "type":"public-key"
}
```

{% hint style="info" %}
The library needs PSR-7 requests. In the example below, we use `nyholm/psr7-server` to get that request.
{% endhint %}

```php
<?php

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

try {
    $publicKeyCredentialSource = $server->loadAndCheckAssertionResponse(
        '_The authenticator response you received…',
        $publicKeyCredentialRequestOptions, // The options you stored during the previous step
        $userEntity,                        // The user entity
        $serverRequest                      // The PSR-7 request
    );
    
    //If everything is fine, this means the user has correctly been authenticated using the
    // authenticator defined in $publicKeyCredentialSource
} catch(\Throwable $exception) {
    // Something went wrong!
}
```

