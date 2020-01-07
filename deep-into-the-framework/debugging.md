# Debugging

If you have troubles during the development of your application or if you want to keep track of every critical/error messages in production, you can use a [PSR-3 compatible logger](https://www.php-fig.org/psr/psr-3/).

## The Easy Way

```php
<?php

use App\Service\MyPsr3Logger;
use Webauthn\Server;

$server = new Server(
    $rpEntity
    $publicKeyCredentialSourceRepository
);

// Set your logging service here
$server->setLogger(new MyPsr3Logger());
```

## The Hard Way

The following classes have an optional constructor parameter $logger that can accept the logging service.

* `Webauthn\AttestationStatement\AttestationObjectLoader`
* `Webauthn\AuthenticatorAssertionResponseValidator`
* `Webauthn\AuthenticatorAttestationResponseValidator`
* `Webauthn\PublicKeyCredentialLoader`
* `Webauthn\Counter\ThrowExceptionIfInvalid`

## The Symfony Way

{% code title="config/packages/webauthn.yaml" %}
```yaml
webauthn:
    logger: App\Service\MyPsr3Logger
```
{% endcode %}

