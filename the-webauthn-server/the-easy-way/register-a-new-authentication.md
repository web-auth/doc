# Authenticator Registration

### Credential Creation Options

Now we want to register a new authenticator and attach it to a user. This step can be done during the creation of a new user account or if the user already exists and you want to add another authenticator.

{% hint style="success" %}
You can attach several authenticators to a user account. It is recommended in case of lost devices or if the user get access on your application using multiple platforms \(smartphone, laptop…\).
{% endhint %}

To register a new authenticator, you need to generate and send a set of options to it. These options defined in a `Webauthn\PublicKeyCredentialCreationOptions` object.

To generate that object, you just need to call the method`generatePublicKeyCredentialCreationOptions` of the `$server` object. This method requires a `Webauthn\PublicKeyCredentialUserEntity` object that represents the user entity to be associated with this new authenticator.

```php
<?php

use Webauthn\PublicKeyCredentialUserEntity;

$userEntity = new PublicKeyCredentialUserEntity(
    'john.doe',
    'ea4e7b55-d8d0-4c7e-bbfa-78ca96ec574c',
    'John Doe'
);

$publicKeyCredentialCreationOptions = $server->generatePublicKeyCredentialCreationOptions(
    $userEntity
);
```

Now send the options to the authenticator using your favorite Javascript framework, library or the example availbale in [the Javascript page](../../pre-requisites/javascript.md).

{% hint style="warning" %}
The variable `$publicKeyCredentialCreationOptions` and `$userEntity` have to be stored somewhere. These are needed during the next step. Usually these values are set in the session or solutions like Redis.
{% endhint %}

### Response Verification

When the authenticator send you the computed response \(i.e. the user touched the button, fingerprint reader, submitted the PIN…\), you can load it and check it.

The authenticator response looks similar to the following example:

```javascript
{
    "id": "LFdoCFJTyB82ZzSJUHc-c72yraRc_1mPvGX8ToE8su39xX26Jcqd31LUkKOS36FIAWgWl6itMKqmDvruha6ywA",
    "rawId": "LFdoCFJTyB82ZzSJUHc-c72yraRc_1mPvGX8ToE8su39xX26Jcqd31LUkKOS36FIAWgWl6itMKqmDvruha6ywA",
    "response": {
        "clientDataJSON": "eyJjaGFsbGVuZ2UiOiJOeHlab3B3VktiRmw3RW5uTWFlXzVGbmlyN1FKN1FXcDFVRlVLakZIbGZrIiwiY2xpZW50RXh0ZW5zaW9ucyI6e30sImhhc2hBbGdvcml0aG0iOiJTSEEtMjU2Iiwib3JpZ2luIjoiaHR0cDovL2xvY2FsaG9zdDozMDAwIiwidHlwZSI6IndlYmF1dGhuLmNyZWF0ZSJ9",
        "attestationObject": "o2NmbXRoZmlkby11MmZnYXR0U3RtdKJjc2lnWEcwRQIgVzzvX3Nyp_g9j9f2B-tPWy6puW01aZHI8RXjwqfDjtQCIQDLsdniGPO9iKr7tdgVV-FnBYhvzlZLG3u28rVt10YXfGN4NWOBWQJOMIICSjCCATKgAwIBAgIEVxb3wDANBgkqhkiG9w0BAQsFADAuMSwwKgYDVQQDEyNZdWJpY28gVTJGIFJvb3QgQ0EgU2VyaWFsIDQ1NzIwMDYzMTAgFw0xNDA4MDEwMDAwMDBaGA8yMDUwMDkwNDAwMDAwMFowLDEqMCgGA1UEAwwhWXViaWNvIFUyRiBFRSBTZXJpYWwgMjUwNTY5MjI2MTc2MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEZNkcVNbZV43TsGB4TEY21UijmDqvNSfO6y3G4ytnnjP86ehjFK28-FdSGy9MSZ-Ur3BVZb4iGVsptk5NrQ3QYqM7MDkwIgYJKwYBBAGCxAoCBBUxLjMuNi4xLjQuMS40MTQ4Mi4xLjUwEwYLKwYBBAGC5RwCAQEEBAMCBSAwDQYJKoZIhvcNAQELBQADggEBAHibGMqbpNt2IOL4i4z96VEmbSoid9Xj--m2jJqg6RpqSOp1TO8L3lmEA22uf4uj_eZLUXYEw6EbLm11TUo3Ge-odpMPoODzBj9aTKC8oDFPfwWj6l1O3ZHTSma1XVyPqG4A579f3YAjfrPbgj404xJns0mqx5wkpxKlnoBKqo1rqSUmonencd4xanO_PHEfxU0iZif615Xk9E4bcANPCfz-OLfeKXiT-1msixwzz8XGvl2OTMJ_Sh9G9vhE-HjAcovcHfumcdoQh_WM445Za6Pyn9BZQV3FCqMviRR809sIATfU5lu86wu_5UGIGI7MFDEYeVGSqzpzh6mlcn8QSIZoYXV0aERhdGFYxEmWDeWIDoxodDQXD2R2YFuP5K65ooYyx5lc87qDHZdjQQAAAAAAAAAAAAAAAAAAAAAAAAAAAEAsV2gIUlPIHzZnNIlQdz5zvbKtpFz_WY-8ZfxOgTyy7f3Ffbolyp3fUtSQo5LfoUgBaBaXqK0wqqYO-u6FrrLApQECAyYgASFYIPr9-YH8DuBsOnaI3KJa0a39hyxh9LDtHErNvfQSyxQsIlgg4rAuQQ5uy4VXGFbkiAt0uwgJJodp-DymkoBcrGsLtkI"
    },
    "type": "public-key"
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
    $publicKeyCredentialSource = $server->loadAndCheckAttestationResponse(
        '_The authenticator response you received…',
        $publicKeyCredentialCreationOptions, // The options you stored during the previous step
        $serverRequest                       // The PSR-7 request
    );
    
    // The user entity and the public key credential source can now be stored using their repository
    // The Public Key Credential Source repository must implement Webauthn\PublicKeyCredentialSourceRepository
    $publicKeyCredentialSourceRepository->saveCredentialSource($publicKeyCredentialSource);
    
    // If you create a new user account, you should also save the user entity
    $userEntityRepository->save($userEntity);
} catch(\Throwable $exception) {
    // Something went wrong!
}
```

