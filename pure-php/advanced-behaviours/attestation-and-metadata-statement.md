# Attestation and Metadata Statement

{% hint style="danger" %}
Disclaimer: you **should not** ask for the Attestation Statement unless you are working on an application that requires a high level of trust (e.g. Banking/Financial Company, Government Agency...).
{% endhint %}

## Receiving Attestation Statement

### Attestation Metadata Repository

First of all, you must prepare an Attestation Metadata Repository. This service will manage all Metadata Statements depending on their sources (local storage or distant service).

Your Metadata Statement Repository must implement the interface `Webauthn\MetadataService\MetadataStatementRepository` that has two methods:

* `findOneByAAGUID(string $aaguid)`: this method retrieves the `MetadataStatement` object with AAGUID. It shall return `null` in case of the absence of the MDS.

{% hint style="warning" %}
The library does not provide any Metadata Statement Repository. It is up to you to select the MDS suitable for your application and store them in your database.
{% endhint %}

There are few steps to acheive. First, you have to add support classes for all attestation statement types into your Attestation Metatdata Manager.

{% hint style="warning" %}
The _Android SafetyNet Attestation Statement_ is a JWT that can be verified by the library, but can also be checked online by hitting the Google API. This method drastically increase the security for the attestation type but requires a [PSR-18 compatible HTTP Client](https://www.php-fig.org/psr/psr-18/) and [an API key](https://developer.android.com/training/safetynet/attestation).
{% endhint %}

{% hint style="warning" %}
For 4.5.0, the `TPMAttestationStatementSupport` class accepts a PSR-20 clock as argument. This argument will be mandatory for 5.0.0.

In the example below, we use `symfony/clock` component.
{% endhint %}

```php
<?php

declare(strict_types=1);

use Cose\Algorithm\Manager;
use Cose\Algorithm\Signature\ECDSA;
use Cose\Algorithm\Signature\EdDSA;
use Cose\Algorithm\Signature\RSA;
use Webauthn\AttestationStatement\AndroidSafetyNetAttestationStatementSupport;
use Webauthn\AttestationStatement\AndroidKeyAttestationStatementSupport;
use Webauthn\AttestationStatement\AttestationStatementSupportManager;
use Webauthn\AttestationStatement\FidoU2FAttestationStatementSupport;
use Webauthn\AttestationStatement\NoneAttestationStatementSupport;
use Webauthn\AttestationStatement\PackedAttestationStatementSupport;
use Webauthn\AttestationStatement\TPMAttestationStatementSupport;
use Webauthn\AttestationStatement\AppleAttestationStatementSupport;
use Symfony\Component\Clock\NativeClock;

//We need a PSR-20 clock
$clock = new NativeClock();

// You normally already do this
$attestationStatementSupportManager = AttestationStatementSupportManager::create();
$attestationStatementSupportManager->add(NoneAttestationStatementSupport::create());

// Additional classes to add
$attestationStatementSupportManager->add(FidoU2FAttestationStatementSupport::create());
$attestationStatementSupportManager->add(AppleAttestationStatementSupport::create());

$androidSafetyNetAttestationStatementSupport = AndroidSafetyNetAttestationStatementSupport::create()
    ->enableApiVerification( $psr18Client, $googleApiKey, $psr17RequestFactory) // Optional
;
$attestationStatementSupportManager->add($androidSafetyNetAttestationStatementSupport);
$attestationStatementSupportManager->add(AndroidKeyAttestationStatementSupport::create());
$attestationStatementSupportManager->add(TPMAttestationStatementSupport::create($clock));

// Cose Algorithm Manager
// The list of algorithm depends on the algorithm list you defined in your options
// You should use at least ES256 and RS256 algorithms that are widely used.
$coseAlgorithmManager = Manager::create();
$coseAlgorithmManager->add(ECDSA\ES256::create());
$coseAlgorithmManager->add(RSA\RS256::create());

$attestationStatementSupportManager->add(PackedAttestationStatementSupport::create($coseAlgorithmManager));
```

Next, you must inject the Metadata Statement Repository to your Attestation Object Loader.

```php
$attestationObjectLoader = AttestationObjectLoader::create($attestationStatementSupportManager);
```

### Credential Creation Options

By default, no Attestation Statement is asked to the Authenticators (type = `none`). To change this behavior, you just have to set the corresponding parameter in the `Webauthn\PublicKeyCredentialCreationOptions` object.

There are 3 conveyance modes available using PHP constants provided by the class `Webauthn\PublicKeyCredentialCreationOptions`:

* `ATTESTATION_CONVEYANCE_PREFERENCE_NONE`: the Relying Party is not interested in authenticator attestation (default)
* `ATTESTATION_CONVEYANCE_PREFERENCE_INDIRECT`: the Relying Party prefers an attestation conveyance yielding verifiable attestation statements, but allows the client to decide how to obtain such attestation statements.
* `ATTESTATION_CONVEYANCE_PREFERENCE_DIRECT`: the Relying Party wants to receive the attestation statement as generated by the authenticator.

#### The Hard Way

```php
<?php

use Webauthn\PublicKeyCredentialCreationOptions;

$publicKeyCredentialCreationOptions = PublicKeyCredentialCreationOptions::create(
    $relyingParty
    $userEntity,
    $challenge,
    $pubKeyCredParams
);
```
