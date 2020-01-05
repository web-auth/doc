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
* ~~Token Binding support~~
* Cose Algorithms
  *  RS1, RS256, RS384, RS512
  *  PS256, PS384, PS512
  *  ES256, ES256K, ES384, ES512
  *  ED25519
* Extensions
  * Supported \(not fully tested\)
  * appid extension

### Compatible Authenticators

The compliance of the framework is ensured by running unit and functional tests during its development.

It is also tested using the official FIDO Alliance testing tools. The status of the compliance tests are [reported in this issue](https://github.com/web-auth/webauthn-framework/issues/67). At the time of writing \(end of Oct. 2019\), the main features and algorithms are supported. Full compliance with the Webatuhn specification is expected by the end of Nov. 2019.

In any case, the framework can already compatible with all authenticators except the one that use ECDAA Attestation format. As this format is very rare at that time, this framework can safely be used in production.

