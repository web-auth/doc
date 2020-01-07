---
description: Overview of the framework
---

# Introduction

Webauthn defines an API enabling the creation and use of strong, attested, scoped, public key-based credentials by web applications, for the purpose of strongly authenticating users.

The complete specification can be found on [the W3C dedicated page](https://www.w3.org/TR/webauthn/).

This framework contains PHP libraries and Symfony bundle to allow developpers to integrate that authentication mechanism into their web applications.

## Class, Constant and Property Names

Naming things may be complicated. Thas’s why the following rule applies on the whole framework: the name of classes, constants and properties as the ones you will find in the specification.

As an example, the [section 5.3.3 “Web Authentication Assertion”](https://www.w3.org/TR/webauthn/#iface-authenticatorassertionresponse) shows an object named `AuthenticatorAssertionResponse` that extends `AuthenticatorResponse` with the following properties:

* `authenticatorData`
* `signature`
* `userHandle`

 You will find [EXACTLY the same structure](https://github.com/web-auth/webauthn-framework/blob/v3.0/src/webauthn/src/AuthenticatorAssertionResponse.php#L21) in the PHP class provided by the library.

## Supported features

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

## Compatible Authenticators

The framework is already compatible with all authenticators except the one that use ECDAA Attestation format.

{% hint style="info" %}
The ECDAA Attestation format is very rare at that time \(January 2020\) thus this framework can safely be used in production.
{% endhint %}

The compliance of the framework is ensured by running unit and functional tests during its development.

It is also tested using the official FIDO Alliance testing tools. The status of the compliance tests are [reported in this issue](https://github.com/web-auth/webauthn-framework/issues/67). At the time of writing \(end of January. 2020\), the main features and algorithms are supported and 99% of the tests pass. Full compliance with the Webatuhn specification is expected in early 2020.

## Support

I bring solutions to your problems and answer your questions.

If you really love that project and the work I have done or if you want I prioritize your issues, then [you can help me out for a couple of🍻 or more](https://github.com/sponsors/Spomky)!

## Contributing

Requests for new features, bug fixed and all other ideas to make this framework useful are welcome.

If you feel comfortable writing code, you could try to fix [opened issues where help is wanted](https://github.com/web-auth/webauthn-framework/issues?q=label%3A%22help+wanted%22) or [those that are easy to fix](https://github.com/web-auth/webauthn-framework/labels/easy-pick).

Do not forget to follow [these best practices](contributing.md).

{% hint style="danger" %}
If you think you have found a security issue, **DO NOT open an issue**. [You MUST submit your issue here](https://gitter.im/Spomky/).
{% endhint %}

