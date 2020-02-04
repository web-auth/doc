---
description: Step-by-step guide for migrating from v2.x to v3.0
---

# From v2.x to v3.0

This project follows the [Semantic Versioning principles](https://semver.org/) and, contrary to upgrade a minor version \(where the middle number changes\) where no difficulty should be encountered, upgrade a major version \(where the first number changes\) is subject to significant modifications.

## Update the libraries <a id="update-the-libraries"></a>

First of all, you have to make sure you are using the last v2.x release \(v2.1.7 at the time of writing\).

## Spot deprecations <a id="spot-deprecations"></a>

Next, you have to verify you don’t use any deprecated class, interface, method or property. If you have PHPUnit tests, [you can easily get the list of deprecation used in your application](https://symfony.com/doc/current/components/phpunit_bridge.html).

{% hint style="danger" %}
This section is not yet finished
{% endhint %}

### General modifications

#### CBOR\Decoder Service Not Needed Anymore

You don't have to inject the `CBOR\Decoder` service anymore. This service is automatically created with the necessary  options.

Example:

```php
use Webauthn\AuthenticatorAssertionResponseValidator;

//Before:
$validator = new AuthenticatorAssertionResponseValidator(
    $publicKeyCredentialSourceRepository,
    $decoder,
    $tokenBindingHandler,
    $extensionOutputCheckerHandler,
    $algorithmManager
);

//After
$validator = new AuthenticatorAssertionResponseValidator(
    $publicKeyCredentialSourceRepository,
    $tokenBindingHandler,
    $extensionOutputCheckerHandler,
    $algorithmManager
);
```

#### Metadata Statement Repository Is Required

You must inject the Metadata Statement Repository to use Metadata Statement types other than `none`.

### Removed methods, functions or properties

* `Webauthn\PublicKeyCredentialSource::createFromPublicKeyCredential()`
* `Webauthn\ CertificateToolbox::checkAttestationMedata()`

## Internal Dependency Changes:

* Added:
  * `league/uri-components ^2.1`
  * `psr/log ^1.1`
  * `sensio/framework-extra-bundle ^5.2`
* Bumped:
  * `league/uri` from `^5.3` to `^6.0`
  * `spomky-labs/cbor-bundle` from `^1.0` to `^2.0`
  * `symfony/*` from `^4.3` to `^5.0`
* Removed:
  * `symfony/http-client`

## Update your Configuration Files <a id="upgrade-the-libraries"></a>

{% hint style="info" %}
To be written
{% endhint %}

## Upgrade the libraries <a id="upgrade-the-libraries"></a>

It is now time to upgrade the libraries. In your composer.json, change all `web-auth/*` dependencies from `v2.x` to `v3.0`. When done, execute `composer update`.

{% hint style="warning" %}
This may also update other dependencies. You can list upgradable libraries by calling `composer outdated`. Please make sure these libraries do not impact your upgrade.
{% endhint %}

## All Modifications In A Row

If you want to see all modifications at once, please [have a look at this page](https://github.com/web-auth/webauthn-framework/compare/v2.1.7...v3.0).

