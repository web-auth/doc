---
description: How to run a basic Webauthn server?
---

# The Easy Way

The easiest way to create a Webauthn Server is to use the class `Webauthn\Server`.

```php
<?php

use Webauthn\Server;
use Webauthn\PublicKeyCredentialRpEntity;

$rpEntity = PublicKeyCredentialRpEntity::create(
    'Webauthn Server',
    'my.domain.com'
);
$publicKeyCredentialSourceRepository = …; //Your repository here. Must implement Webauthn\PublicKeyCredentialSourceRepository

$server = Server::create(
    $rpEntity
    $publicKeyCredentialSourceRepository
);
```

That’s it!

You can now [register a new authenticator](register-a-new-authentication.md) or [authenticate your users](user-authentication.md).

