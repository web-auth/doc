---
description: Step-by-step guide for migrating from 4.x to 5.0
---

# From 4.x to 5.0

{% hint style="info" %}
This page is subject to changes as the version 5.0.0 is not available at the time of writing.
{% endhint %}

This project follows the [Semantic Versioning principles](https://semver.org) and, contrary to upgrade a minor version (where the middle number changes) where no difficulty should be encountered, upgrade a major version (where the first number changes) is subject to significant modifications.

## Update the libraries <a href="#update-the-libraries" id="update-the-libraries"></a>

First of all, you have to make sure you are using the last 4.x release (4.8.0 at the time of writing).

In addition, you have to make sure you are using PHP `8.3+`.

## Spot deprecations <a href="#spot-deprecations" id="spot-deprecations"></a>

Next, you have to verify you don’t use any deprecated class, interface, method or property. If you have PHPUnit tests, [you can easily get the list of deprecation used in your application](https://symfony.com/doc/current/components/phpunit\_bridge.html).

#### PSR-20 Clock

In previous versions, the classes that requires time used the PHP time function directly. It is now required to use a PSR-20 Clock implementation and pass it to the classes.

* Webauthn\MetadataService\CertificateChain\PhpCertificateChainValidator

For version 3.2.0+ and the Symfony Bundle, an internal implementation service named `jose.internal_clock` existed and is removed.

### Token Binding

All references to token binding are deprecated. This functionality is not supported anymore as removed from the latest Webauthn spectification versions.

### ECDAA

All references to the ECDAA Attestation Statement type are deprecated. This functionality is not supported anymore as removed from the latest Webauthn spectification versions.

### Webauthn\AuthenticatorSelectionCriteria

* Constant `AuthenticatorSelectionCriteria::RESIDENT_KEY_REQUIREMENT_NONE`: please use `AuthenticatorSelectionCriteria::RESIDENT_KEY_REQUIREMENT_NO_PREFERENCE` instead

### Webauthn\CertificateToolbox

This class is now deprecated. Please use `Webauthn\MetadataService\CertificateChainChecker\PhpCertificateChainValidator` instead or a class that implements `Webauthn\MetadataService\CertificateChain\CertificateChainValidator`.

### Webauthn\PublicKeyCredentialLoader

This class is removed in 5.0. You should use Symfony Serializer or create a dedicated serializer using `Webauthn\Denormalizer\WebauthnSerializerFactory`.

### Webauthn\PublicKeyCredentialSourceRepository

This interface is deprecated and removed. There is no replacement as it became useless for the library. The Symfony bundle uses its own interface `Webauthn\Bundle\Repository\Webauthn\Bundle\Repository` you are asked to use in the Symfony context.

### Symfony Http Client

The PSR-17 and PSR-18 are not supported anymore. The library uses Symfony Http Client instead. A class is provided to help you to continue using PSR-\* compatible libraries: `Webauthn\MetadataService\Psr18HttpClient`. This class is very basic and can be enhanced or overridden at will.

### Events

The following events are removed in favor of events located in the library namespace:

* `Webauthn\Bundle\Event\AuthenticatorAssertionResponseValidationFailedEvent`
* `Webauthn\Bundle\Event\AuthenticatorAssertionResponseValidationSucceededEvent`
* `Webauthn\Bundle\Event\AuthenticatorAttestationResponseValidationFailedEvent`
* `Webauthn\Bundle\Event\AuthenticatorAttestationResponseValidationSucceededEvent`

### Services

The following services are removed:

* `webauthn.cose.algoritm.*` (because of a typo)
* `Webauthn\PublicKeyCredentialLoader`
* `Webauthn\PublicKeyCredentialSourceRepository`
* `Webauthn\TokenBinding\IgnoreTokenBindingHandler`
* `Webauthn\TokenBinding\SecTokenBindingHandler`
* `Webauthn\TokenBinding\TokenBindingNotSupportedHandler`

## Dependency Changes:

* Added:
  * `symfony/clock`
  * `symfony/serializer`
  * `symfony/property-access`
  * `symfony/property-info`
  * `phpdocumentor/reflection-docblock`
* Bumped:
  * `PHP`: `>=8.3`
  * `symfony/*`: `^7.0`
* Removed:
  * `lcobucci/clock`

## Configuration Files <a href="#upgrade-the-libraries" id="upgrade-the-libraries"></a>

The following options are removed:

* `webauthn.http_message_factory`
* `webauthn.token_binding_support_handler`
* `webauthn.creation_profiles[x].attachment_mode`

## Upgrade the libraries <a href="#upgrade-the-libraries" id="upgrade-the-libraries"></a>

When deprecations are removed, you can upgrade the libraries. In your composer.json, change all `web-auth/*` dependencies from `^4.x` to `^5.0`. When done, execute `composer update`.

{% hint style="warning" %}
This may also update other dependencies. You can list upgradable libraries by calling `composer outdated`. Please make sure these libraries do not impact your upgrade.
{% endhint %}

## All Modifications In A Row

If you want to see all modifications at once, please [have a look at this page](https://github.com/web-auth/webauthn-framework/compare/4.0.x...5.0.x).
