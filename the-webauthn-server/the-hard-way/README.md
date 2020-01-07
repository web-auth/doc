# The Hard Way

You will need the following components before loading or verifying the data:

* [The Public Key Credential Source Repository](../../pre-requisites/credential-souce-repository.md)
* [A token binding handler](../../deep-into-the-framework/token-binding.md)
* An Attestation Statement Support Manager and at least one Attestation Statement Support object
* An Attestation Object Loader
* A Public Key Credential Loader
* An Authenticator Attestation Response Validator
* An Extension Output Checker Handler

That’s a lot off classes! But don’t worry, as their configuration is the same for all your application, you just have to set them once. Let see all of these in the next sections.

## Public Key Credential Source Repository

The Public Key Credential Source Repository must implement `Webauthn\PublicKeyCredentialSourceRepository`. It will retrieve the credential source and update them when needed.

You can implement the required methods the way you want: Doctrine ORM, file storage… as mentioned on [the dedicated page](../../pre-requisites/credential-souce-repository.md).

## Token Binding Handler

The token binding handler is a service that will verify if the token binding set in the device response corresponds to the one set in the request.

Please refer to [the dedicated page](../../deep-into-the-framework/token-binding.md).

## Attestation Statement Support Manager

Every Creation Responses contain an Attestation Statement. This attestation contains data regarding the authenticator depending on several factors such as its manufacturer and model,what you asked in the options, the capabilities of the browser or what the user allowed.

{% hint style="info" %}
With Firefox for example, the user may refuse to send information about the security token for privacy reasons.
{% endhint %}

Hereafter the types of attestations you can have:

* `none`: no attestation is provided
* `fido-u2f`: for non-FIDO2 compatible devices \(old U2F security token\)
* `packed`: generaly used by 
* `android key`: commonly used by old or disconnected Android devices
* `android safety net`: for new Android devices like smartphones
* `trusted platform module`: for devices with built-in secutrity chips

{% hint style="danger" %}
All these attestation types are supported, but you should only use the `none` one unless you plan to use the [Attestation and Metadata Statement](../../deep-into-the-framework/attestation-and-metadata-statement.md).
{% endhint %}

{% hint style="warning" %}
The _Android SafetyNet Attestation Statement_ is a JWT that can be verified by the library, but can also be checked online by hitting the Google API. This method drastically increase the security for the attestation type but requires a [PSR-18 compatible HTTP Client](https://www.php-fig.org/psr/psr-18/) and [an API key](https://developer.android.com/training/safetynet/attestation).
{% endhint %}

```php
<?php

declare(strict_types=1);

use Webauthn\AttestationStatement\AttestationStatementSupportManager;
use Webauthn\AttestationStatement\NoneAttestationStatementSupport;

// The manager will receive data to load and select the appropriate 
$attestationStatementSupportManager = new AttestationStatementSupportManager();

// The none type
$attestationStatementSupportManager->add(new NoneAttestationStatementSupport());
```

## Attestation Object Loader

This object will load the Attestation statements received from the devices. It will need the Attestation Statement Support Manager created above.

```php
<?php

declare(strict_types=1);

use Webauthn\AttestationStatement\AttestationObjectLoader;

$attestationObjectLoader = new AttestationObjectLoader($attestationStatementSupportManager);
```

## Public Key Credential Loader

This object will load the Public Key using from the Attestation Object.

```php
<?php

declare(strict_types=1);

use Webauthn\PublicKeyCredentialLoader;

$publicKeyCredentialLoader = new PublicKeyCredentialLoader($attestationObjectLoader);
```

## Extension Output Checker Handler

If you use extensions, you may need to check the value returned by the security devices. This behaviour is handled by an Extension Output Checker Manager.

```php
<?php

declare(strict_types=1);

use Webauthn\AuthenticationExtensions\ExtensionOutputCheckerHandler;

$extensionOutputCheckerHandler = new ExtensionOutputCheckerHandler();
```

You can add as many extension checker as you want. Each extension checker must implement `Webauthn\AuthenticationExtensions\ExtensionOutputChecker` and throw a `Webauthn\AuthenticationExtensions\ExtensionOutputError` in case of an error.

## Authenticator Attestation Response Validator

This object is what you will directly use when receiving Attestation Responses \(authenticator registration\).

```php
<?php

declare(strict_types=1);

use Webauthn\AuthenticatorAttestationResponseValidator;

$authenticatorAttestationResponseValidator = new AuthenticatorAttestationResponseValidator(
    $attestationStatementSupportManager,
    $publicKeyCredentialSourceRepository,
    $tokenBindingHandler,
    $extensionOutputCheckerHandler
);
```

## Authenticator Assertion Response Validator

This object is what you will directly use when receiving Assertion Responses \(user authentication\).

```php
<?php

declare(strict_types=1);

use Webauthn\AuthenticatorAssertionResponseValidator;

$authenticatorAssertionResponseValidator = new AuthenticatorAssertionResponseValidator(
    $publicKeyCredentialSourceRepository,  // The Credential Repository service
    $tokenBindingHandler,                  // The token binding handler
    $extensionOutputCheckerHandler,        // The extension output checker handler
    $coseAlgorithmManager                  // The COSE Algorithm Manager  
);
```

