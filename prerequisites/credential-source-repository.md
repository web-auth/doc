---
description: Authenticator details and how to manage them
---

# Credential Source

## Credential Source Class

After the registration of an authenticator, you will get a Public Key Credential Source object. It contains all the credential data needed to perform user authentication and much more:

* The credential ID,
* The public key,
* The credential type (`public_key`),
* The transports (`USB`, `NFC`, `BLE`, `internal`),
* The attestation type,
* The trust path,
* The authenticator AAGUID,
* The user handle (i.e. the user ID)
* The authenticator counter,
* Other UI data
* ...

## Credential Source Repository

Since 4.6.0 and except if you use the Symfony bundle, there is no interface to implement or abstract class to extend so that it should be easy to integrate it in your application.

{% hint style="success" %}
Whatever database you use (MySQL, pgSQL…), it is not necessary to create relationships between your users and the Credential Sources.
{% endhint %}
