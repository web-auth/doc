# Webauthn Server

To launch a Webauthn server, you will need the following components:

* [A Public Key Credential Source Repository](the-hard-way.md#public-key-credential-source-repository)
* [A token binding handler](the-hard-way.md#token-binding-handler)
* [An Attestation Statement Support Manager](the-hard-way.md#attestation-statement-support-manager)
* [At least one Attestation Statement Support object](the-hard-way.md#supported-attestation-statement-types)
* [An Attestation Object Loader](the-hard-way.md#attestation-object-loader)
* [A Public Key Credential Loader](the-hard-way.md#public-key-credential-loader)
* [An Extension Output Checker Handler](the-hard-way.md#extension-output-checker-handler)
* [An Algorithm Manager](the-hard-way.md#undefined)
* [An Authenticator Attestation Response Validator](the-hard-way.md#authenticator-attestation-response-validator)
* [An Authenticator Assertion Response Validator](the-hard-way.md#authenticator-assertion-response-validator)

That’s a lot off classes! But don’t worry, as their configuration is the same for all your application, you just have to set them once. Let’s see all of these in the next sections.

## Public Key Credential Source Repository

The Public Key Credential Source Repository must implement `Webauthn\PublicKeyCredentialSourceRepository`. It will retrieve the credential source and update them when needed.

You can implement the required methods the way you want: Doctrine ORM, file storage…

```php
$publicKeyCredentialSourceRepository = ...; //Instantiate your repository
```

## Token Binding Handler

The token binding handler is a service that will verify if the token binding set in the device response corresponds to the one set in the request.

At the time of writing, we recommend to ignore this feature. Please refer to [the dedicated page](../webauthn-in-a-nutshell/token-binding.md) for more information.

```php
use Webauthn\TokenBinding\IgnoreTokenBindingHandler;

$tokenBindingHandler = IgnoreTokenBindingHandler::create();
```

## Attestation Statement Support Manager

Every Creation Responses contain an Attestation Statement. This attestation contains data regarding the authenticator depending on several factors such as its manufacturer and model, what you asked in the options, the capabilities of the browser or what the user allowed.

{% hint style="info" %}
The user may refuse to send information about the security token for privacy reasons.Hereafter the types of attestations you may have:
{% endhint %}

### Supported Attestation Statement Types

The following attestation types are supported. Note that you should only use the `none` one unless you have specific needs described in [the dedicated page](../webauthn-in-a-nutshell/attestation-and-metadata-statement.md).

* `none`: no attestation is provided.
* `fido-u2f`: for non-FIDO2 compatible devices (old U2F security tokens).
* `packed`: generally used by authenticators with limited resources (e.g., secure elements). It uses a very compact but still extensible encoding method.
* `android key`: commonly used by old or disconnected Android devices.
* `android safety net`: for new Android devices like smartphones.
* `trusted platform module`: for devices with built-in security chips.
* `apple`: for Apple devices

```php
<?php

declare(strict_types=1);

use Webauthn\AttestationStatement\AttestationStatementSupportManager;
use Webauthn\AttestationStatement\NoneAttestationStatementSupport;

// The manager will receive data to load and select the appropriate 
$attestationStatementSupportManager = AttestationStatementSupportManager::create()
    ->add(NoneAttestationStatementSupport::create())
;
```

## Attestation Object Loader

This object will load the Attestation statements received from the devices. It will need the Attestation Statement Support Manager created above.

```php
<?php

declare(strict_types=1);

use Webauthn\AttestationStatement\AttestationObjectLoader;

$attestationObjectLoader = AttestationObjectLoader::create(
    $attestationStatementSupportManager
);
```

## Public Key Credential Loader

This object will load the Public Key using from the Attestation Object.

```php
<?php

declare(strict_types=1);

use Webauthn\PublicKeyCredentialLoader;

$publicKeyCredentialLoader = PublicKeyCredentialLoader::create(
    $attestationObjectLoader
);
```

## Extension Output Checker Handler

If you use extensions, you may need to check the value returned by the security devices. This behaviour is handled by an Extension Output Checker Manager.

```php
<?php

declare(strict_types=1);

use Webauthn\AuthenticationExtensions\ExtensionOutputCheckerHandler;

$extensionOutputCheckerHandler = ExtensionOutputCheckerHandler::create();
```

You can add as many extension checkers as you want. Each extension checker must implement `Webauthn\AuthenticationExtensions\ExtensionOutputChecker` and throw a `Webauthn\AuthenticationExtensions\ExtensionOutputError` in case of an error.

More about that [in this page](advanced-behaviours/extensions.md).

## Algorithm Manager

The Webauthn data verification is based on cryptographic signatures and thus you need to provide cryptographic algorithms to perform those checks.

We recommend the use of the following algorithms to cover all types of Authenticators. Feel free to adapt this list depending on your needs.

```php
<?php

declare(strict_types=1);

use Cose\Algorithm\Manager;
use Cose\Algorithm\Signature\ECDSA\ES256;
use Cose\Algorithm\Signature\ECDSA\ES256K;
use Cose\Algorithm\Signature\ECDSA\ES384;
use Cose\Algorithm\Signature\ECDSA\ES512;
use Cose\Algorithm\Signature\EdDSA\Ed256;
use Cose\Algorithm\Signature\EdDSA\Ed512;
use Cose\Algorithm\Signature\RSA\PS256;
use Cose\Algorithm\Signature\RSA\PS384;
use Cose\Algorithm\Signature\RSA\PS512;
use Cose\Algorithm\Signature\RSA\RS256;
use Cose\Algorithm\Signature\RSA\RS384;
use Cose\Algorithm\Signature\RSA\RS512;

$algorithmManager = Manager::create()
    ->add(
        ES256::create(),
        ES256K::create(),
        ES384::create(),
        ES512::create(),

        RS256::create(),
        RS384::create(),
        RS512::create(),

        PS256::create(),
        PS384::create(),
        PS512::create(),

        Ed256::create(),
        Ed512::create(),
    )
;
```

## Authenticator Attestation Response Validator

This object is what you will directly use when receiving Attestation Responses (authenticator registration).

```php
<?php

declare(strict_types=1);

use Webauthn\AuthenticatorAttestationResponseValidator;

$authenticatorAttestationResponseValidator = AuthenticatorAttestationResponseValidator::create(
    $attestationStatementSupportManager,
    $publicKeyCredentialSourceRepository,
    $tokenBindingHandler,
    $extensionOutputCheckerHandler
);
```

## Authenticator Assertion Response Validator

This object is what you will directly use when receiving Assertion Responses (user authentication).

```php
<?php

declare(strict_types=1);

use Webauthn\AuthenticatorAssertionResponseValidator;

$authenticatorAssertionResponseValidator = AuthenticatorAssertionResponseValidator::create(
    $publicKeyCredentialSourceRepository,  // The Credential Repository service
    $tokenBindingHandler,                  // The token binding handler
    $extensionOutputCheckerHandler,        // The extension output checker handler
    $algorithmManager                      // The COSE Algorithm Manager  
);
```
