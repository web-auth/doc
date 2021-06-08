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
* **Anonymization CA** \(`AnonCA`\):  Authenticators use an Anonymization CA, which dynamically generates per-credential attestation certificates such that the attestation statements presented to Relying Parties do not provide uniquely identifiable information.
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

Your Metadata Statement Repository must implement the interface `Webauthn\MetadataService\MetadataStatementRepository` that has a unique method `findOneByAAGUID(string $aaguid)`.

#### Basic Repository Implementation.

The library `web-auth/metadata-service` provides a concrete class with basic support for local and distant statements with caching system: `Webauthn\MetadataService\SimpleMetadataStatementRepository`

```php
use Webauthn\MetadataService\MetadataStatementRepository:
use Symfony\Component\Cache\Adapter\FilesystemAdapter;
use Webauthn\MetadataService\SingleMetadata;

$myMetadataStatementRepository = new SimpleMetadataStatementRepository(
    new FilesystemAdapter('webauthn') // We use filesystem caching in this example
);

// We add a local matadata statement (adapted from the Yubico website)
$myMetadataStatementRepository->addSingleStatement('yubico', new SingleMetadata(
    '{"description": "Yubico U2F Root CA Serial 457200631","aaguid": "f8a011f3-8c0a-4d15-8006-17111f9edc7d","protocolFamily": "fido2","attestationRootCertificates": ["MIIDHjCCAgagAwIBAgIEG0BT9zANBgkqhkiG9w0BAQsFADAuMSwwKgYDVQQDEyNZdWJpY28gVTJGIFJvb3QgQ0EgU2VyaWFsIDQ1NzIwMDYzMTAgFw0xNDA4MDEwMDAwMDBaGA8yMDUwMDkwNDAwMDAwMFowLjEsMCoGA1UEAxMjWXViaWNvIFUyRiBSb290IENBIFNlcmlhbCA0NTcyMDA2MzEwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC/jwYuhBVlqaiYWEMsrWFisgJ+PtM91eSrpI4TK7U53mwCIawSDHy8vUmk5N2KAj9abvT9NP5SMS1hQi3usxoYGonXQgfO6ZXyUA9a+KAkqdFnBnlyugSeCOep8EdZFfsaRFtMjkwz5Gcz2Py4vIYvCdMHPtwaz0bVuzneueIEz6TnQjE63Rdt2zbwnebwTG5ZybeWSwbzy+BJ34ZHcUhPAY89yJQXuE0IzMZFcEBbPNRbWECRKgjq//qT9nmDOFVlSRCt2wiqPSzluwn+v+suQEBsUjTGMEd25tKXXTkNW21wIWbxeSyUoTXwLvGS6xlwQSgNpk2qXYwf8iXg7VWZAgMBAAGjQjBAMB0GA1UdDgQWBBQgIvz0bNGJhjgpToksyKpP9xv9oDAPBgNVHRMECDAGAQH/AgEAMA4GA1UdDwEB/wQEAwIBBjANBgkqhkiG9w0BAQsFAAOCAQEAjvjuOMDSa+JXFCLyBKsycXtBVZsJ4Ue3LbaEsPY4MYN/hIQ5ZM5p7EjfcnMG4CtYkNsfNHc0AhBLdq45rnT87q/6O3vUEtNMafbhU6kthX7Y+9XFN9NpmYxr+ekVY5xOxi8h9JDIgoMP4VB1uS0aunL1IGqrNooL9mmFnL2kLVVee6/VR6C5+KSTCMCWppMuJIZII2v9o4dkoZ8Y7QRjQlLfYzd3qGtKbw7xaF1UsG/5xUb/Btwb2X2g4InpiB/yt/3CpQXpiWX/K4mBvUKiGn05ZsqeY1gx4g0xLBqcU9psmyPzK+Vsgw2jeRQ5JlKDyqE0hebfC1tvFu0CCrJFcw=="]}',
    false // The statement is not base64 encoded
));
```

{% hint style="warning" %}
The example above is very limited and will only allow authenticators manufactured by Yubico to be registered. Make sure to add more sources of Metadata Statements to accept authenticators from other manufacturers.
{% endhint %}

When the repository is ready, you must inject it to your server.

#### The Easy Way

You just have to inject the Metadata Statement Repository to your `Server` class.

```php
<?php

use Webauthn\Server;
use Webauthn\PublicKeyCredentialRpEntity;

...

// Before v3.3
$server = new Server(
    $rpEntity,
    $publicKeyCredentialSourceRepository,
    $myMetadataStatementRepository        // Inject your new service here
);

// Or v3.3+
$server = new Server(
    $rpEntity,
    $publicKeyCredentialSourceRepository
);
$server->setMetadataStatementRepository($myMetadataStatementRepository); // Inject your new service here
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
use Webauthn\AttestationStatement\AppleAttestationStatementSupport;

// You normally already do this
$attestationStatementSupportManager = new AttestationStatementSupportManager();
$attestationStatementSupportManager->add(new NoneAttestationStatementSupport());

// Additional classes to add
$attestationStatementSupportManager->add(new FidoU2FAttestationStatementSupport());
$attestationStatementSupportManager->add(new AppleAttestationStatementSupport());
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

There are 3 conveyance modes available using PHP constants provided by the class `Webauthn\PublicKeyCredentialCreationOptions`:

* `ATTESTATION_CONVEYANCE_PREFERENCE_NONE`: the Relying Party is not interested in authenticator attestation \(default\)
* `ATTESTATION_CONVEYANCE_PREFERENCE_INDIRECT`: the Relying Party prefers an attestation conveyance yielding verifiable attestation statements, but allows the client to decide how to obtain such attestation statements.
* `ATTESTATION_CONVEYANCE_PREFERENCE_DIRECT`: the Relying Party wants to receive the attestation statement as generated by the authenticator.

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
    $relayingParty
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

