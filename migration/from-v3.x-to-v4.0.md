---
description: Step-by-step guide for migrating from v3.x to v4.0
---

# From v3.x to v4.0

This project follows the [Semantic Versioning principles](https://semver.org/) and, contrary to upgrade a minor version \(where the middle number changes\) where no difficulty should be encountered, upgrade a major version \(where the first number changes\) is subject to significant modifications.

## Update the libraries <a id="update-the-libraries"></a>

First of all, you have to make sure you are using the last v3.x release \(v3.3.4 at the time of writing\).

In addition, you have to make sure you are using PHP `8.0+`.

## Spot deprecations <a id="spot-deprecations"></a>

Next, you have to verify you don’t use any deprecated class, interface, method or property. If you have PHPUnit tests, [you can easily get the list of deprecation used in your application](https://symfony.com/doc/current/components/phpunit_bridge.html).

{% hint style="danger" %}
This section is not yet finished
{% endhint %}

## Dependency Changes:

* Bumped:
  * `symfony/psr-http-message-bridge` minimal version is now `^2.0`
* Removed:
  * `sensio/framework-extra-bundle`

## Update your Configuration Files <a id="upgrade-the-libraries"></a>

As the bundle `sensio/framework-extra-bundle` is not required anymore, the associated configuration may become useless.

## Upgrade the libraries <a id="upgrade-the-libraries"></a>

It is now time to upgrade the libraries. In your composer.json, change all `web-auth/*` dependencies from `v3.x` to `v4.0`. When done, execute `composer update`.

{% hint style="warning" %}
This may also update other dependencies. You can list upgradable libraries by calling `composer outdated`. Please make sure these libraries do not impact your upgrade.
{% endhint %}

## All Modifications In A Row

If you want to see all modifications at once, please [have a look at this page](https://github.com/web-auth/webauthn-framework/compare/v3.3.4...v4.0).

