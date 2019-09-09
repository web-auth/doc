# The Symfony Way

Symfony is a very popular framework and an official bundle is provided in the package `web-auth/webauthn-symfony-bundle`.

{% hint style="info" %}
If you use Laravel, you may be intersted in [this project: https://github.com/asbiin/laravel-webauthn](https://github.com/asbiin/laravel-webauthn)
{% endhint %}

If you are using Symfony Flex then the bundle will automatically be installed. Otherwise you need to add it in your `AppKernel.php` file:

{% code-tabs %}
{% code-tabs-item title="src/AppKernel.php" %}
```php
<?php

public function registerBundles()
{
    $bundles = [
        // ...
        new Webauthn\Bundle\WebauthnBundle(),
    ];
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Entities

At the moment, only Doctrine is supported, however there is no technical constraint to allow other data storage systems.

* [With Doctrine](entities-with-doctrine.md)

### Configuration

{% hint style="info" %}
To be written
{% endhint %}

### Available Services

{% hint style="info" %}
To be written
{% endhint %}

### Firewall

{% hint style="info" %}
To be written
{% endhint %}

