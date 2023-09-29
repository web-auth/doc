# Bundle Installation

This framework provides a Symfony bundle that will help you to use the components within your Symfony application.

{% hint style="info" %}
If you use Laravel, you may be interested in [this project: https://github.com/asbiin/laravel-webauthn](https://github.com/asbiin/laravel-webauthn)
{% endhint %}

## Installation

### With Symfony Flex

```shell
composer require web-auth/webauthn-symfony-bundle
```

{% hint style="info" %}
As the recipes are third party ones not officially supported by Symfony, you will be asked to execute the recipe or not. When applied, the recipes will prompt a message such as `WebAuthn Framework is ready.`
{% endhint %}

### Without Symfony Flex

If you don't use Symfony Flex, you must register the bundle and route manually.

{% code title="config/bundles.php" %}
```php
<?php

return [
    //...
    Webauthn\Bundle\WebauthnBundle::class => ['all' => true],
];
```
{% endcode %}

{% code title="config/routes/webauthn.php" %}
```php
<?php

declare(strict_types=1);

use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->import('.', 'webauthn');
};
```
{% endcode %}

## Repositories

The first steps are:

* The creation of your [Credential Source repository](entities-with-doctrine.md)
* The creation of your [User Entity repository](entities-with-doctrine-1.md).

You may also want to configure the other options offered by the bundle. Please refer to the [configuration references](configuration-references.md).

## Firewall

Now you have a fully configured bundle, you can protect your routes and manage the user registration and authenticatin through the [Symfony Firewall](firewall.md).
