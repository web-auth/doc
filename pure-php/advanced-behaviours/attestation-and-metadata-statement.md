# Attestation and Metadata Statement

{% hint style="danger" %}
**Important Notice:** The request for an Attestation Statement is reserved for applications necessitating a high degree of reliability, such as those operated by banking or financial institutions, government agencies, etc. Exercise caution and ensure your application's context requires such a level of trust before proceeding.
{% endhint %}

## Attestation Statement Support Manager

There are few steps to acheive. First, you have to add support classes for all attestation statement types into your Attestation Metatdata Manager.

{% hint style="warning" %}
For 4.5.0, the `TPMAttestationStatementSupport` class accepts a PSR-20 clock as argument. This argument will be mandatory for 5.0.0.

In the example below, we use `symfony/clock` component.
{% endhint %}

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Cose\Algorithm\Manager;
use Cose\Algorithm\Signature\ECDSA;
use Cose\Algorithm\Signature\EdDSA;
use Cose\Algorithm\Signature\RSA;
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
{% endcode %}

{% hint style="warning" %}
The Android SafetyNet Attestation API is deprecated. Full turndown is planned in June 2024. More information at [https://developer.android.com/training/safetynet/deprecation-timeline](https://developer.android.com/training/safetynet/deprecation-timeline)

It is not described on this page, but available in the previous versions of the documentation.
{% endhint %}

## Metadata Statement Repository

Then, you must prepare an Metadata Statement Repository. This service will manage all Metadata Statements depending on their sources (local storage or distant service).

Your Metadata Statement Repository must implement the interface `Webauthn\MetadataService\MetadataStatementRepository` that has only one method:

* `findOneByAAGUID(string $aaguid)`: this method retrieves the `MetadataStatement` object with AAGUID. It shall return `null` in case of the absence of the MDS.

{% hint style="warning" %}
The library does not provide any Metadata Statement Repository. It is up to you to select the MDS suitable for your application and store them in your database.
{% endhint %}

## Status Report Repository

To prevent the use of rogue MDS or deprecated by the manufacturers, Status Reports may be used. The Status Report Repository is a simple service that implements the interface `Webauthn\MetadataService\StatusReportRepository` with a unique method:

* `findStatusReportsByAAGUID(string $aaguid)`: this method returns a list of Status Reports for the given AAGUID.

The verification's success or failure depends on the state reported in the most recent Status Report.

## Certificate Chain Validator

When an attestation statement is received, a certificate chain is constructed. This chain combines certificates from the Metadata Service (MDS), which are trusted, with those provided by the authenticator, which are untrusted.

The Certificate Chain Validator is responsible for this task. The library provides the class `Webauthn\MetadataService\CertificateChain\PhpCertificateChainValidator`  that requires a Symfony Http Client ([for CRL verification](https://en.wikipedia.org/wiki/Certificate\_revocation\_list)) and a PSR-20 Clock.

{% code lineNumbers="true" %}
```php
<?php

use Webauthn\MetadataService\CertificateChain\PhpCertificateChainValidator;

$certificateChainValidator = PhpCertificateChainValidator::create(
    $httpClient,
    $clock
);
```
{% endcode %}

## Ceremony Step Manager Factory

The services described above must be set to the [Ceremony Step Manager Factory](../input-validation.md). The CSM you will create will verifiy the attestation statement sent by the authenticator.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\CeremonyStep\CeremonyStepManagerFactory;

$csmFactory = new CeremonyStepManagerFactory();
$csmFactory->setAttestationStatementSupportManager($attestationStatementSupportManager);
$csmFactory->enableMetadataStatementSupport(
    $metadataStatementRepository,
    $statusReportRepository,
    $certificateChainValidator,
);
```
{% endcode %}

## Requesting Attestation Statement

By default, no Attestation Statement is asked to the Authenticators (type = `none`). To change this behavior, you just have to set the corresponding parameter in the `Webauthn\PublicKeyCredentialCreationOptions` object.

There are 3 conveyance modes available using PHP constants provided by the class `Webauthn\PublicKeyCredentialCreationOptions`:

* `ATTESTATION_CONVEYANCE_PREFERENCE_NONE`: the Relying Party is not interested in authenticator attestation (default)
* `ATTESTATION_CONVEYANCE_PREFERENCE_INDIRECT`: the Relying Party prefers an attestation conveyance yielding verifiable attestation statements, but allows the client to decide how to obtain such attestation statements.
* `ATTESTATION_CONVEYANCE_PREFERENCE_DIRECT`: the Relying Party wants to receive the attestation statement as generated by the authenticator.

{% code lineNumbers="true" %}
```php
<?php

use Webauthn\PublicKeyCredentialCreationOptions;

$publicKeyCredentialCreationOptions = PublicKeyCredentialCreationOptions::create(
    $relyingParty,
    $userEntity,
    $challenge,
    attestation: PublicKeyCredentialCreationOptions::ATTESTATION_CONVEYANCE_PREFERENCE_DIRECT,
);
```
{% endcode %}
