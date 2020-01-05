# The Symfony Way

Symfony is a very popular framework and an official bundle is provided in the package `web-auth/webauthn-symfony-bundle`.

{% hint style="info" %}
If you use Laravel, you may be intersted in [this project: https://github.com/asbiin/laravel-webauthn](https://github.com/asbiin/laravel-webauthn)
{% endhint %}

If you are using Symfony Flex then the bundle will automatically be installed. Otherwise you need to add it in your `AppKernel.php` file:

{% code title="src/AppKernel.php" %}
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
{% endcode %}

### Entities

At the moment, only Doctrine is supported, however there is no technical constraint to allow other data storage systems.

* [With Doctrine](entities-with-doctrine.md)

### Configuration

The minimal configuration requires the[ user repository](../../pre-requisites/user-entity-repository.md) and the [pk credential source repository](../../pre-requisites/credential-souce-repository.md).

{% code title="app/config/webauthn.yaml" %}
```yaml
webauthn:
    credential_repository: 'App\Repository\PublicKeyCredentialSourceRepository'
    user_repository: 'App\Repository\PublicKeyCredentialUserEntityRepository'
```
{% endcode %}

Now you may want to:

* [Register your first authenticators](register-authenticators.md),
* [Authenticate your users](firewall.md).

