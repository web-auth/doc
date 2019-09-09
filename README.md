# Introduction

## Overview of the framework

Webauthn defines an API enabling the creation and use of strong, attested, scoped, public key-based credentials by web applications, for the purpose of strongly authenticating users.

This framework contains PHP libraries and Symfony bundle to allow developpers to integrate that authentication mechanism into their web applications.

### Supported features

* Attestation Types
  * Empty
  * Basic
  * Self
  * Private CA
  * ~~Elliptic Curve Direct Anonymous Attestation \(ECDAA\)~~
* Attestation Formats
  *   FIDO U2F
  * Packed
  * TPM
  * Android Key
  * Android Safetynet
* ~~TokenBinding support~~
* Cose Algorithms
  *  RS1, RS256, RS384, RS512
  *  PS256, PS384, PS512
  *  ES256, ES256K, ES384, ES512
  *  ED25519
* Extensions
  * Supported \(not fully tested\)
  * appid extension

