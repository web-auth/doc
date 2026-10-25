# Bundle installation

This framework provides a Symfony bundle that will help you to use the components within your Symfony application.

!!! info

    Implementations for other frameworks and web applications exist.

    As an example, if you use Laravel, you may be interested in [this project: https://github.com/asbiin/laravel-webauthn](https://github.com/asbiin/laravel-webauthn)

## Installation

### With Symfony Flex

```shell
composer require web-auth/webauthn-symfony-bundle
```

!!! info

    As the recipes are third party ones not officially supported by Symfony, you will be asked to execute the recipe or not. When applied, the recipes will prompt a message such as `WebAuthn Framework is ready.`

### Without Symfony Flex

If you don't use Symfony Flex, you must register the bundle and route manually.

```php title="config/bundles.php" linenums="1"
<?php

return [
    //...
    Webauthn\Bundle\WebauthnBundle::class => ['all' => true],
];
```

```php title="config/routes/webauthn.php" linenums="1"
<?php

declare(strict_types=1);

use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return function (RoutingConfigurator $routes) {
    $routes->import('.', 'webauthn');
};
```

## Repositories

The first steps are:

* The creation of your [Credential Record Repository](credential-record-repository.md)
* The creation of your [User Entity Repository](user-entity-repository.md).

You may also want to configure the other options offered by the bundle. Please refer to the [configuration references](configuration-references.md).

## Firewall

Now you have a fully configured bundle, you can protect your routes and manage the user registration and authentication through the [Symfony Firewall](firewall.md).
