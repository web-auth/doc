# Register/Authenticate Programmatically

In some circumstances, you may need to register a new authenticator for a user e.g. when adding a new authenticator or when an administrator acts as another user to replace a lost device.

It is possible to perform both ceremonies programmatically.

## Register Authenticators

### Generate Options

You can directly create a `Webauthn\PublicKeyCredentialCreationOptions` object, but it is better to take advantages of the [creation profiles](./#configuration-of-the-creation-profiles) using the `Webauthn\Bundle\Service\PublicKeyCredentialCreationOptionsFactory` service.

If the user already registered one or more authenticators, there is a risk that the same authenticator is registered twice. To prevent that, you can pass a list of Public Key Descriptor IDs.

```php
use Webauthn\Bundle\Service\PublicKeyCredentialCreationOptionsFactory;
use Webauthn\PublicKeyCredentialDescriptor;
use Webauthn\PublicKeyCredentialSource;
use Webauthn\PublicKeyCredentialUserEntity;


// The User Entity
$userEntity = new PublicKeyCredentialUserEntity(
    'john.doe',
    'ea4e7b55-d8d0-4c7e-bbfa-78ca96ec574c',
    'John Doe'
);

// We gather all registered authenticators for this user
$registeredAuthenticators = $container
    ->get(YourPublicKeyCredentialSourceRepository::class)
    ->findAllForUserEntity($userEntity)
;

// We don’t need the Credential Sources, just the associated Descriptors
$excludedCredentials = array_map(
    static function (PublicKeyCredentialSource $credential): PublicKeyCredentialDescriptor {
        return $credential->getPublicKeyCredentialDescriptor();
    },
    $registeredAuthenticators
);

// We generate the options
// This object will be needed later and should be stored in session
$publicKeyCredentialCreationOptions = $container
    ->get(PublicKeyCredentialCreationOptionsFactory::class)
    ->create(
        'acme',              // The name of the profile
        $userEntity,         // The user entity
        $excludedCredentials // The excluded authenticators
    )
;
```

Next, you can send the options to the authenticator \(can be serialized into a JSON object\).

### Load And Check Responses

When you received the authenticator response, you can load it and check it.

```php
use Webauthn\AuthenticatorAttestationResponse;
use Webauthn\PublicKeyCredentialLoader;
use Webauthn\AuthenticatorAttestationResponseValidator;

$publicKeyCredential = $container->get(PublicKeyCredentialLoader::class)->load($authenticatorResponse);
$response = $publicKeyCredential->getResponse();
if (!$response instanceof AuthenticatorAttestationResponse) {
   // Redirect or throw an exception
}

$publicKeyCredentialSource = $container->get(AuthenticatorAssertionResponseValidator::class)->check(
   $response,                           // The authenticator response
   $publicKeyCredentialCreationOptions, // The object we generated earlier
   $request                             // The request (PSR-7 object!)
);

//You can save the user entity (if not already done) and the new Public Key Credential Source object
$container->get(YourPublicKeyCredentialSourceRepository::class)->saveCredentialSource($publicKeyCredentialSource);
```

## Authenticate Users

### Generate Options

As for the registration process, you can directly create a `Webauthn\PublicKeyCredentialRequestOptions` object or use [request profiles](./#request-profiles) and the `Webauthn\Bundle\Service\PublicKeyCredentialRequestOptionsFactory` service.

```php
use Webauthn\Bundle\Service\PublicKeyCredentialCreationOptionsFactory;
use Webauthn\PublicKeyCredentialDescriptor;
use Webauthn\PublicKeyCredentialSource;
use Webauthn\PublicKeyCredentialUserEntity;


// The User Entity
$userEntity = new PublicKeyCredentialUserEntity(
    'john.doe',
    'ea4e7b55-d8d0-4c7e-bbfa-78ca96ec574c',
    'John Doe'
);

// We gather all registered authenticators for this user
$registeredAuthenticators = $container
    ->get(YourPublicKeyCredentialSourceRepository::class)
    ->findAllForUserEntity($userEntity)
;

// We don’t need the Credential Sources, just the associated Descriptors
$allowedCredentials = array_map(
    static function (PublicKeyCredentialSource $credential): PublicKeyCredentialDescriptor {
        return $credential->getPublicKeyCredentialDescriptor();
    },
    $registeredAuthenticators
);

// We generate the options
// This object will be needed later and should be stored in session
$publicKeyCredentialCreationOptions = $container
    ->get(PublicKeyCredentialCreationOptionsFactory::class)
    ->create(
        'acme',             // The name of the profile
        $allowedCredentials // The authenticators allowed to be used
    )
;
```

Next, you can send the options to the authenticator \(can be serialized into a JSON object\).

### Load And Check Responses

When you received the authenticator response, you can load it and check it.

```php
use Webauthn\AuthenticatorAssertionResponse;
use Webauthn\PublicKeyCredentialLoader;
use Webauthn\AuthenticatorAssertionResponseValidator;

$publicKeyCredential = $container->get(PublicKeyCredentialLoader::class)->load($authenticatorResponse);
$response = $publicKeyCredential->getResponse();
if (!$response instanceof AuthenticatorAssertionResponse) {
    // Redirect or throw an exception
}

$publicKeyCredentialSource = $container->get(AuthenticatorAssertionResponseValidator::class)->check(
    $publicKeyCredential->getRawId(),   // The ID of the credential used by the user
    $response,                          // The authenticator response
    $publicKeyCredentialRequestOptions, // The object we generated earlier
    $request,                           // The request (PSR-7 object!)
    $userEntity->getId()                // The user entity ID
);

```

If no exception is thrown, the user is correctly authenticated using the credential source returned by the method `check`.

