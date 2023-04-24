# Metadata Statement

{% hint style="danger" %}
Disclaimer: you **should not** ask for the Attestation Statement unless you are working on an application that requires a high level of trust (e.g. Banking/Financial Company, Government Agency...).
{% endhint %}

## Attestation Statement

During the Attestation Ceremony (i.e. the registration of the authenticator), you can ask for the Attestation Statement of the authenticator. The Attestation Statements have one of the following types:

* **None** (`none`): no Attestation Statement is provided
* **Basic Attestation** (`basic`)**:** Authenticator’s attestation key pair is specific to an authenticator model.
* **Surrogate Basic Attestation (or Self Attestation -** `self`**)**: Authenticators that have no specific attestation key use the credential private key to create the attestation signature
* **Attestation CA** (`AttCA`): Authenticators are based on a Trusted Platform Module (TPM). They can generate multiple attestation identity key pairs (AIK) and requests an Attestation CA to issue an AIK certificate for each.
* **Anonymization CA** (`AnonCA`): Authenticators use an Anonymization CA, which dynamically generates per-credential attestation certificates such that the attestation statements presented to Relying Parties do not provide uniquely identifiable information.

## Metadata Statement

The Metadata Statements are issued by the manufacturers of the authenticators. These statements contain details about the authenticators (supported algorithms, biometric capabilities...) and all the necessary information to verify the Attestation Statements generated during the attestation ceremony.

There are several possible sources to get these Metadata Statements. The main source is the [FIDO Alliance Metadata Service](https://fidoalliance.org/metadata) that allows fetching statements on-demand, but some of them may be provided by other means.

{% hint style="danger" %}
The FIDO Alliance Metadata Service provides a limited number of Metadata Statements. It is mandatory to get the statement from the manufacturer of your authenticators otherwise the Attestation Statement won't be verified and the Attestation Ceremony will fail.
{% endhint %}
