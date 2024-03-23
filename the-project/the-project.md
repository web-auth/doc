---
description: Overview of the framework
---

# What is Webauthn?

Webauthn defines an API enabling the creation and use of strong, attested, scoped, public key-based credentials by web applications, for the purpose of strongly authenticating users.

The complete specification can be found on [the W3C dedicated page](https://www.w3.org/TR/webauthn-2/).

This framework contains PHP libraries and Symfony bundle to allow developers to integrate that authentication mechanism into their web applications.

## Class, Constant and Property Names

Naming things may be complicated. That’s why the following rule applies on the whole framework: the name of classes, constants and properties are identical to the ones you will find in the specification.

As an example, the [section 5.2.2 “Web Authentication Assertion”](https://www.w3.org/TR/webauthn-2/#iface-authenticatorassertionresponse) shows an object named `AuthenticatorAssertionResponse` that extends `AuthenticatorResponse` with the following properties:

* `authenticatorData`
* `signature`
*   `userHandle`

    You will find [EXACTLY the same structure](https://github.com/web-auth/webauthn-framework/blob/v3.0/src/webauthn/src/AuthenticatorAssertionResponse.php#L21) in the PHP class provided by the library.

## Supported features

* Attestation Types
  * Empty
  * Basic
  * Self
  * Private CA
  * Anonymization CA

{% hint style="info" %}
Note that _Elliptic Curve Direct Anonymous Attestation (ECDAA)_ is deprecated and not supported
{% endhint %}

* Attestation Formats
  * FIDO U2F
  * Packed
  * TPM
  * Android Key
  * Android Safetynet
  * Apple
* Cose Algorithms
  * RS1, RS256, RS384, RS512
  * PS256, PS384, PS512
  * ES256, ES256K, ES384, ES512
  * ED25519
* Extensions
  * Supported (not fully tested)
  * appid extension (compatibility with FIDO U2F authenticator

{% hint style="info" %}
The _Token Binding support_ feature is now deprecated as not part of the latest specification version
{% endhint %}

## Compatible Authenticators

The framework is already compatible with all authenticators.

The compliance of the framework is ensured by running unit and functional tests during its development.

It is also tested using the official FIDO Alliance testing tools. The status of the compliance tests are [reported in this issue](https://github.com/web-auth/webauthn-framework/issues/67). At the time of writing (January 2023, all features and algorithms are supported and 100% of the tests pass.

{% hint style="info" %}
See [https://github.com/herrjemand/awesome-webauthn/pull/61](https://github.com/herrjemand/awesome-webauthn/pull/61) and [https://github.com/herrjemand/awesome-webauthn#server-libs](https://github.com/herrjemand/awesome-webauthn#server-libs)
{% endhint %}

## Support

I bring solutions to your problems and answer your questions.

If you really love that project, and the work I have done or if you want I prioritize your issues, then [you can help me out for a couple of🍻 or more](https://github.com/sponsors/Spomky)!

## Contributing

Requests for new features, bug fixed and all other ideas to make this framework useful are welcome.

If you feel comfortable writing code, you could try to fix [opened issues where help is wanted](https://github.com/web-auth/webauthn-framework/issues?q=label%3A%22help+wanted%22) or [those that are easy to fix](https://github.com/web-auth/webauthn-framework/labels/easy-pick).

Do not forget to follow [these best practices](contributing.md).

{% hint style="danger" %}
If you think you have found a security issue, **DO NOT open an issue**. [You MUST submit your issue here](https://github.com/web-auth/webauthn-framework/security).
{% endhint %}
