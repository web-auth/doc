---
description: How to install the library or the Symfony bundle?
---

# Installation

This framework contains several sub-packages that you don’t necessarily need. It is highly recommended to install what you need and not the whole framework.

The preferred way to install the library you need is to use composer:

```bash
composer require web-auth/webauthn-lib
```

Hereafter the dependency tree:

* `web-auth/webauthn-lib`: this is the core library. This package can be used in any PHP project or within any popular framework \(Laravel, CakePHP…\)
* `web-auth/webauthn-symfony-bundle`: this is a Symfony bundle that ease the integration of this authentication mechanism in your Symfony project.
* `web-auth/conformance-toolset`: this component helps you to verify your application is compliant with the specification. It is meant to be used with the FIDO Alliance Tools. You usually don’t need it.

The core library also depends on `web-auth/cose-lib` and `web-auth/metadata-service`. What are these dependencies?

`web-auth/cose-lib` contains several cipher algorithms and COSE key support to verify the digital signatures sent by the authenticators during the creation and authentication ceremonies. These algorithms are compliant with the [RFC8152](https://tools.ietf.org/html/rfc8152). This library can be used by any other PHP projects. At the moment only signature algorithms are available, but it is planned to add encryption algorithms.

`web-auth/metadata-service` provides classes to support the [Fido Alliance Metadata Service](https://fidoalliance.org/metadata/). If you plan to use Attestation Statements during the creation ceremony, this service is mandatory. Please note that Attestation Statements decreases the user privacy as they may leak data that allow to identify a specific user. **The use of Attestation Statements and this service are generally not recommended unless you REALLY need this information**. This library can also be used by any other PHP projects.

## Framework and Dependency Sizes

The total size of the core package is approximately 760ko. Hereafter the detail for each component:

* `web-auth/cose-lib`: 85ko
* `web-auth/metadata-service`: 81ko
* `web-auth/webauthn-lib`: 207ko
* `web-auth/webauthn-symfony-bundle`: 385ko
* `web-auth/conformance-toolset`: N/A

The total size of the core package + the direct dependencies is approximately 1.7Mo.

