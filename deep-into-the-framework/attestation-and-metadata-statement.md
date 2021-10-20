# Attestation and Metadata Statement

{% hint style="danger" %}
Disclaimer: you **should not** ask for the Attestation Statement unless you are working on an application that requires a high level of trust \(e.g. Banking/Financial Company, Government Agency...\).
{% endhint %}

## Attestation Statement

During the Attestation Ceremony \(i.e. the registration of the authenticator\), you can ask for the Attestation Statement of the authenticator. The Attestation Statements have one of the following types:

* **None** \(`none`\): no Attestation Statement is provided
* **Basic Attestation** \(`basic`\)**:** Authenticator’s attestation key pair is specific to an authenticator model.
* **Surrogate Basic Attestation \(or Self Attestation -** `self`**\)**: Authenticators that have no specific attestation key use the credential private key to create the attestation signature
* **Attestation CA** \(`AttCA`\): Authenticators are based on a Trusted Platform Module \(TPM\). They can generate multiple attestation identity key pairs \(AIK\) and requests an Attestation CA to issue an AIK certificate for each.
* **Elliptic Curve based Direct Anonymous Attestation** \(`ECDAA`\): Authenticator receives direct anonymous attestation \(DAA\) credentials from a single DAA-Issuer. These DAA credentials are used along with blinding to sign the attested credential data.

## Metadata Statement

The Metadata Statements are issued by the manufacturers of the authenticators. These statements contain details about the authenticators \(supported algorithms, biometric capabilities...\) and all the necessary information to verify the Attestation Statements generated during the attestation ceremony.

There are several possible sources to get these Metadata Statements. The main source is the [FIDO Alliance Metadata Service](https://fidoalliance.org/metadata) that allows fetching statements on-demand, but some of them may be provided by other means.

{% hint style="danger" %}
The FIDO Alliance Metadata Service provides a limited number of Metadata Statements. It is mandatory to get the statement from the manufacturer of your authenticators otherwise the Attestation Statement won't be verified and the Attestation Ceremony will fail.
{% endhint %}

## Receiving Attestation Statement

### Attestation Metadata Repository

First of all, you must prepare an Attestation Metadata Repository. This service will manage all Metadata Statements depending on their sources \(local storage or distant service\).

Your Metadata Statement Repository must implement the interface `Webauthn\MetadataService\MetadataStatementRepository` that has two methods:

* `findOneByAAGUID(string $aaguid)`: this method retrieves the `MetadataStatement` object with AAGUID. It shall return `null` in case of the absence of the MDS.
* `findStatusReportsByAAGUID(string $aaguid)`: retrieves all `StatusReport` objects. These objects show the MDS compliance history.

{% hint style="warning" %}
The library does not provide any Metadata Statement Repositroy. It is up to you to select the MDS suitable for your application and store them in your database.
{% endhint %}

#### The Easy Way

You just have to inject the Metadata Statement Repository to your `Server` class.

```php
<?php

use Webauthn\Server;
use Webauthn\PublicKeyCredentialRpEntity;

...

$server = new Server(
    $rpEntity
    $publicKeyCredentialSourceRepository,
    $myMetadataStatementRepository        // Inject your new service here
);
```

#### The Hard Way

There are few steps to acheive. First, you have to add support classes for all attestation statement types into your Attestation Metatdata Manager.

{% hint style="warning" %}
The _Android SafetyNet Attestation Statement_ is a JWT that can be verified by the library, but can also be checked online by hitting the Google API. This method drastically increase the security for the attestation type but requires a [PSR-18 compatible HTTP Client](https://www.php-fig.org/psr/psr-18/) and [an API key](https://developer.android.com/training/safetynet/attestation).
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

// You normally already do this
$attestationStatementSupportManager = new AttestationStatementSupportManager();
$attestationStatementSupportManager->add(new NoneAttestationStatementSupport());

// Additional classes to add
$attestationStatementSupportManager->add(new FidoU2FAttestationStatementSupport());
$attestationStatementSupportManager->add(new AndroidSafetyNetAttestationStatementSupport());
$attestationStatementSupportManager->add(new AndroidKeyAttestationStatementSupport(
    $psr18Client,         // Can be null if you don’t want to use the Google API
    $googleApiKey,        // Can be null if you don’t want to use the Google API
    $psr17RequestFactory  // Can be null if you don’t want to use the Google API
));
$attestationStatementSupportManager->add(new TPMAttestationStatementSupport());

// Cose Algorithm Manager
// The list of algorithm depends on the algorithm list you defined in your options
// You should use at least ES256 and RS256 algorithms that are widely used.
$coseAlgorithmManager = new Manager();
$coseAlgorithmManager->add(new ECDSA\ES256());
$coseAlgorithmManager->add(new RSA\RS256());

$attestationStatementSupportManager->add(new PackedAttestationStatementSupport($coseAlgorithmManager));
```

Next, you must inject the Metadata Statement Repository to your Attestation Object Loader.

```php
$attestationObjectLoader = new AttestationObjectLoader($attestationStatementSupportManager, $metadataStatementRepository);
```

#### The Symfony Way

With Symfony, you must enable this feature to enable all the metadata types.

{% code title="config/packages/webauthn.yaml" %}
```yaml
webauthn:
    metadata_service:
        enabled: true
        repository: 'App\Repository\MyMetadataStatementRepository'
```
{% endcode %}

You can set the Google API key for the Android SafetyNet Attestation Statement support with the following configuration:

```yaml
webauthn:
    android_safetynet:
        http_client: 'my.psr18.http.client'
        request_factory: 'my.psr17.request_factory'
```

If you have some troubles when validating Android SafetyNet Attestation Statement, this may be caused by the leeway of the server clocks or the age of the statement. You can modify the default values as follows:

```yaml
webauthn:
    android_safetynet:
        max_age: 60000 # in milliseconds. Default set to 60000 = 1 min
        leeway: 2000 # in milliseconds. Default set to 0
```

{% hint style="warning" %}
The modification of these parameters is not recommended. You should try to sync your server clock first.
{% endhint %}

### Credential Creation Options

By default, no Attestation Statement is asked to the Authenticators \(type = `none`\). To change this behavior, you just have to set the corresponding parameter in the `Webauthn\PublicKeyCredentialCreationOptions` object.

There are 4 conveyance modes available using PHP constants provided by the class `Webauthn\PublicKeyCredentialCreationOptions`:

* `ATTESTATION_CONVEYANCE_PREFERENCE_NONE`: the Relying Party is not interested in authenticator attestation \(default\)
* `ATTESTATION_CONVEYANCE_PREFERENCE_INDIRECT`: the Relying Party prefers an attestation conveyance yielding verifiable attestation statements, but allows the client to decide how to obtain such attestation statements.
* `ATTESTATION_CONVEYANCE_PREFERENCE_DIRECT`: the Relying Party wants to receive the attestation statement as generated by the authenticator.
* `ATTESTATION_CONVEYANCE_PREFERENCE_ENTERPRISE`: the Relying Party wants to receive an attestation statement that may include uniquely identifying information. This is intended for controlled deployments within an enterprise where the organization wishes to tie registrations to specific authenticators.

{% hint style="info" %}
`ATTESTATION_CONVEYANCE_PREFERENCE_ENTERPRISE` comes from the current draft version of the Webauthn specification. This value may not be supported by all browsers.
{% endhint %}

#### The Easy Way

```php
<?php

use Webauthn\PublicKeyCredentialCreationOptions;

$publicKeyCredentialCreationOptions = $server->generatePublicKeyCredentialCreationOptions(
    $userEntity,
    PublicKeyCredentialCreationOptions::ATTESTATION_CONVEYANCE_PREFERENCE_DIRECT,
);
```

#### The Hard Way

```php
<?php

use Webauthn\PublicKeyCredentialCreationOptions;

$publicKeyCredentialCreationOptions = new PublicKeyCredentialCreationOptions(
    $relyingParty
    $userEntity,
    $challenge,
    $pubKeyCredParams,
    $timeout, 
    $excludeCredentials,
    $authenticatorSelection,
    PublicKeyCredentialCreationOptions::ATTESTATION_CONVEYANCE_PREFERENCE_DIRECT
);
```

#### The Symfony Way

{% code title="config/packages/webauthn.yaml" %}
```yaml
webauthn:
    credential_repository: ...
    user_repository: ...
    creation_profiles:
        acme:
            attestation_conveyance: !php/const Webauthn\PublicKeyCredentialCreationOptions::ATTESTATION_CONVEYANCE_PREFERENCE_DIRECT
            rp:
                name: 'My application'
                id: 'example.com'
```
{% endcode %}

