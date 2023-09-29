---
description: Step-by-step guide for migrating from 4.x to 5.0
---

# From 4.x to 5.0

{% hint style="info" %}
THis page is subject to changes as the version 5.0.0 is not available at the time of writing.
{% endhint %}

This project follows the [Semantic Versioning principles](https://semver.org) and, contrary to upgrade a minor version (where the middle number changes) where no difficulty should be encountered, upgrade a major version (where the first number changes) is subject to significant modifications.

## Update the libraries <a href="#update-the-libraries" id="update-the-libraries"></a>

First of all, you have to make sure you are using the last 4.x release (4.6.0 at the time of writing).

In addition, you have to make sure you are using PHP `8.2+`.

## Spot deprecations <a href="#spot-deprecations" id="spot-deprecations"></a>

Next, you have to verify you don’t use any deprecated class, interface, method or property. If you have PHPUnit tests, [you can easily get the list of deprecation used in your application](https://symfony.com/doc/current/components/phpunit\_bridge.html).

### Token Binding

All references to token binding are deprecated. This functionality is not supported anymore as removed from the latest Webauthn spectification versions.

### ECDAA

All references to the ECDAA Attestation Statement type are deprecated. This functionality is not supported anymore as removed from the latest Webauthn spectification versions.

### Webauthn\AuthenticatorSelectionCriteria

* Constant `AuthenticatorSelectionCriteria::RESIDENT_KEY_REQUIREMENT_NONE`: please use `AuthenticatorSelectionCriteria::RESIDENT_KEY_REQUIREMENT_NO_PREFERENCE` instead
* Method&#x20;

### Webauthn\CertificateToolbox

This class is now deprecated. Please use `Webauthn\MetadataService\CertificateChainChecker\PhpCertificateChainValidator` instead or a class that implements `Webauthn\MetadataService\CertificateChain\CertificateChainValidator`.

### Webauthn\PublicKeyCredentialSourceRepository



## Dependency Changes:

* Added:
  * `symfony/clock`: `^6.3`
* Bumped:
  * `PHP`: `>=8.2`
  * `symfony/*`: `^6.3`
* Removed:
  * `lcobucci/clock`

## Configuration Files <a href="#upgrade-the-libraries" id="upgrade-the-libraries"></a>

No modification required.

## Upgrade the libraries <a href="#upgrade-the-libraries" id="upgrade-the-libraries"></a>

When deprecations are removed, you can upgrade the libraries. In your composer.json, change all `web-auth/*` dependencies from `^4.x.y` to `^5.0.0`. When done, execute `composer update`.

{% hint style="warning" %}
This may also update other dependencies. You can list upgradable libraries by calling `composer outdated`. Please make sure these libraries do not impact your upgrade.
{% endhint %}

## All Modifications In A Row

If you want to see all modifications at once, please [have a look at this page](https://github.com/web-auth/webauthn-framework/compare/4.0.x...5.0.x).
