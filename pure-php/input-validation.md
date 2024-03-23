# Input Validation

The loaded data needs to be verified. The library will perform several actions to make sure the input you received is valid. This verification process is performed by a Ceremony Step Manager (CSM). The Webauthn Specification distinguish two types of ceremonies[ described in this page](../webauthn-in-a-nutshell/ceremonies.md).

## Ceremony Step Manager Factory

To facilitate the creation of the CSM, a default factory is included. This factory requires no external services to function.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\CeremonyStep\CeremonyStepManagerFactory;

$csmFactory = new CeremonyStepManagerFactory();

$creationCSM = $csmFactory->creationCeremony();
$requestCSM = $csmFactory->requestCeremony();
```
{% endcode %}

{% hint style="info" %}
You can customize its behavior to fit the specific needs of your application by modifying the provided factory. Please refer to the dedicated pages for more information.

* [Counter Checker](advanced-behaviours/authenticator-counter.md)
* [Attestation Statement Support Manager](advanced-behaviours/attestation-and-metadata-statement.md)
* [Extension Output Checker Handler](advanced-behaviours/extensions.md)
* [Algorithm Manager](advanced-behaviours/authenticator-algorithms.md)
* Certification Chain Validator (page to be written)
{% endhint %}

These CSM services are meant to be used by Response Validators. On a similar way, there are two types of validators:

* Authenticator Attestation Response Validator: used during the creation ceremony
* Authenticator Assertion Response Validator: used during the request ceremony

## Response Validators

The Authenticator Attestation Response Validator and Authenticator Assertion Response Validator services are directly used when receiving Authenticator Responses in order to [register authenticators](authenticator-registration.md) or [authenticate users](authenticate-your-users.md).

{% hint style="info" %}
All null values correspond to deprecated parameters. They will be removed in 5.0.0
{% endhint %}

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\AuthenticatorAttestationResponseValidator;
use Webauthn\AuthenticatorAssertionResponseValidator;

$authenticatorAttestationResponseValidator = AuthenticatorAttestationResponseValidator::create(
    null, //Deprecated
    null, //Deprecated
    null, //Deprecated
    null, //Deprecated
    null, //Deprecated
    $creationCSM
);
$authenticatorAssertionResponseValidator = AuthenticatorAssertionResponseValidator::create(
    null, //Deprecated
    null, //Deprecated
    null, //Deprecated
    null, //Deprecated
    null, //Deprecated
    $requestCSM
);

// Also valid
$authenticatorAttestationResponseValidator = AuthenticatorAttestationResponseValidator::create(
    ceremonyStepManager: $creationCSM
);
$authenticatorAssertionResponseValidator = AuthenticatorAssertionResponseValidator::create(
    ceremonyStepManager: $requestCSM
);
```
{% endcode %}
