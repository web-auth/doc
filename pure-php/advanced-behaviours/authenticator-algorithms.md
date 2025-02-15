# Authenticator Algorithms

The Webauthn data verification is based on cryptographic signatures and thus you need to provide cryptographic algorithms to perform those checks.

The following algorithms are required in most situations:

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Cose\Algorithm\Manager;
use Cose\Algorithm\Signature\ECDSA\ES256;
use Cose\Algorithm\Signature\RSA\RS256;

$algorithmManager = Manager::create()
    ->add(
        ES256::create(),
        RS256::create()
    )
;
```
{% endcode %}

{% hint style="info" %}
The order is important. By adding `ES256` first, the relyaing party prefers an `ES256` credential. Browsers are eager to satisfy preferences.
{% endhint %}

The complete list of supported algorithms:

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Cose\Algorithm\Manager;
use Cose\Algorithm\Signature\ECDSA\ES256;
use Cose\Algorithm\Signature\ECDSA\ES256K;
use Cose\Algorithm\Signature\ECDSA\ES384;
use Cose\Algorithm\Signature\ECDSA\ES512;
use Cose\Algorithm\Signature\EdDSA\Ed256;
use Cose\Algorithm\Signature\EdDSA\Ed512;
use Cose\Algorithm\Signature\RSA\PS256;
use Cose\Algorithm\Signature\RSA\PS384;
use Cose\Algorithm\Signature\RSA\PS512;
use Cose\Algorithm\Signature\RSA\RS256;
use Cose\Algorithm\Signature\RSA\RS384;
use Cose\Algorithm\Signature\RSA\RS512;

$algorithmManager = Manager::create()
    ->add(
        ES256::create(),
        ES256K::create(),
        ES384::create(),
        ES512::create(),

        RS256::create(),
        RS384::create(),
        RS512::create(),

        PS256::create(),
        PS384::create(),
        PS512::create(),

        Ed256::create(),
        Ed512::create(),
    )
;
```
{% endcode %}

The algorithm manager can be injected to your Ceremony Step Manager Factory.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\CeremonyStep\CeremonyStepManagerFactory;

$csmFactory = new CeremonyStepManagerFactory();
$csmFactory->setAlgorithmManager($algorithmManager);
```
{% endcode %}
