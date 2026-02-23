---
description: How to install the library or the Symfony bundle?
---

# Installation

This framework contains several sub-packages that you don’t necessarily need. It is highly recommended to install what you need and not the whole framework.

The preferred way to install the library you need is to use composer:

```bash
composer require web-auth/webauthn-lib
```

If you use Symfony Framework, you may be interested in the bundle and, optionally,  the Stimulus component.

```sh
composer require web-auth/webauthn-symfony-bundle web-auth/webauthn-stimulus
```

## Requirements

* **PHP**: 8.2 or higher
* **Symfony** (for the bundle): 6.4, 7.x, or 8.0+
* **Extensions**: json, openssl
