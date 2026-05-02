---
description: How to install the library or the Symfony bundle?
---

# Installation

This framework contains several sub-packages that you don’t necessarily need. It is highly recommended to install what you need and not the whole framework.

The preferred way to install the library you need is to use composer:

```bash
composer require web-auth/webauthn-lib
```

If you use Symfony Framework, you may be interested in the bundle:

```sh
composer require web-auth/webauthn-symfony-bundle
```

For the Stimulus controllers (frontend integration), pull them straight from npm via AssetMapper or your favourite bundler — see [Symfony UX Installation](../symfony-ux/installation.md) for details:

```bash
php bin/console importmap:require @web-auth/webauthn-stimulus
```

{% hint style="warning" %}
The legacy PHP package `web-auth/webauthn-stimulus` is deprecated since v5.3.0 and will be removed in 6.0.0. Install the controllers via npm instead.
{% endhint %}

## Requirements

* **PHP**: 8.2 or higher
* **Symfony** (for the bundle): 6.4, 7.x, or 8.0+
* **Extensions**: json, openssl
