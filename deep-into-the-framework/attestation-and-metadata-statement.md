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

There are several possible sources to get theses Metadata Statements. The main source is the [FIDO Alliance Metadata Service](https://fidoalliance.org/metadata) that allows to fetch statements on-demand, but some of them may be provided by other means.

{% hint style="danger" %}
The FIDO Alliance Metadata Service provides a limited number of Metadata Statements. It is mandatory to get the statement from the manufacturer of your authenticators otherwise the Attestation Statement won't be verified and the Attestation Ceremony will fail. 
{% endhint %}

## Receiving Attestation Statement

### Attestation Metadata Repository

First of all, you must prepare an Attestation Metadata Repository. This service will manage all Metadata Statements depending on there sources \(local storage or distant service\).

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

{% hint style="info" %}
To be written
{% endhint %}

#### The Symfony Way

With Symfony, every source of Metadata Statement is configured in the application configuration. It will be automatically injected to the services.

{% code-tabs %}
{% code-tabs-item title="config/packages/webauthn.yaml" %}
```yaml
webauthn:
    metadata_service:
        http_client: ... # An HTTP client for distant sources
        request_factory: # PSR-17 request factory
        services: # Services compatible with the MDS Specification
            fido_alliance:
                uri: 'https://mds2.fidoalliance.org'
                additional_query_string_values:
                    token: '--ACCESS-TOKEN--' # We need to set the access token in the query string for this service
        distant_single_statements:
            solo: # A statement provided by Solo (https://solokeys.com/)
                uri: 'https://raw.githubusercontent.com/solokeys/solo/2.1.0/metadata/Solo-FIDO2-CTAP2-Authenticator.json'
                additional_headers: ~
        from_data: # Single statements from local data
            yubico:
                data: '{"description": "Yubico U2F Root CA Serial 457200631","aaguid": "f8a011f3-8c0a-4d15-8006-17111f9edc7d","protocolFamily": "fido2","attestationRootCertificates": ["MIIDHjCCAgagAwIBAgIEG0BT9zANBgkqhkiG9w0BAQsFADAuMSwwKgYDVQQDEyNZdWJpY28gVTJGIFJvb3QgQ0EgU2VyaWFsIDQ1NzIwMDYzMTAgFw0xNDA4MDEwMDAwMDBaGA8yMDUwMDkwNDAwMDAwMFowLjEsMCoGA1UEAxMjWXViaWNvIFUyRiBSb290IENBIFNlcmlhbCA0NTcyMDA2MzEwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC/jwYuhBVlqaiYWEMsrWFisgJ+PtM91eSrpI4TK7U53mwCIawSDHy8vUmk5N2KAj9abvT9NP5SMS1hQi3usxoYGonXQgfO6ZXyUA9a+KAkqdFnBnlyugSeCOep8EdZFfsaRFtMjkwz5Gcz2Py4vIYvCdMHPtwaz0bVuzneueIEz6TnQjE63Rdt2zbwnebwTG5ZybeWSwbzy+BJ34ZHcUhPAY89yJQXuE0IzMZFcEBbPNRbWECRKgjq//qT9nmDOFVlSRCt2wiqPSzluwn+v+suQEBsUjTGMEd25tKXXTkNW21wIWbxeSyUoTXwLvGS6xlwQSgNpk2qXYwf8iXg7VWZAgMBAAGjQjBAMB0GA1UdDgQWBBQgIvz0bNGJhjgpToksyKpP9xv9oDAPBgNVHRMECDAGAQH/AgEAMA4GA1UdDwEB/wQEAwIBBjANBgkqhkiG9w0BAQsFAAOCAQEAjvjuOMDSa+JXFCLyBKsycXtBVZsJ4Ue3LbaEsPY4MYN/hIQ5ZM5p7EjfcnMG4CtYkNsfNHc0AhBLdq45rnT87q/6O3vUEtNMafbhU6kthX7Y+9XFN9NpmYxr+ekVY5xOxi8h9JDIgoMP4VB1uS0aunL1IGqrNooL9mmFnL2kLVVee6/VR6C5+KSTCMCWppMuJIZII2v9o4dkoZ8Y7QRjQlLfYzd3qGtKbw7xaF1UsG/5xUb/Btwb2X2g4InpiB/yt/3CpQXpiWX/K4mBvUKiGn05ZsqeY1gx4g0xLBqcU9psmyPzK+Vsgw2jeRQ5JlKDyqE0hebfC1tvFu0CCrJFcw=="]}'
```
{% endcode-tabs-item %}
{% endcode-tabs %}

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

{% code-tabs %}
{% code-tabs-item title="config/packages/webauthn.yaml" %}
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
{% endcode-tabs-item %}
{% endcode-tabs %}



