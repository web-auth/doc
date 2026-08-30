# Authenticator Algorithms

The WebAuthn data verification is based on cryptographic signatures and thus you need to provide cryptographic algorithms to perform those checks.

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
The order is important. By adding `ES256` first, the relying party prefers an `ES256` credential. Browsers are eager to satisfy preferences.
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

## Fully-Specified Algorithm Identifiers (RFC 9864)

{% hint style="info" %}
**New in v5.4.0**
{% endhint %}

RFC 9864 introduced fully-specified COSE algorithm identifiers, which WebAuthn Level 3 integrated: `-9` (ESP256), `-51` (ESP384), `-52` (ESP512) and `-19` (Ed25519). Within WebAuthn they designate exactly the same algorithms as `-7` (ES256), `-35` (ES384), `-36` (ES512) and `-8` (EdDSA) respectively.

The specification makes them `NOT RECOMMENDED` in `pubKeyCredParams` for backward compatibility, so the default parameters keep the legacy identifiers. An authenticator remains free to return a credential public key that uses one of the new ones, and the Relying Party must be able to verify it. Since v5.4.0:

* an application asking for `ES256` also accepts a credential using `ESP256`, and the other way around, for the four pairs above;
* the built-in algorithm manager, used when none is configured, contains `ESP256` next to `ES256`;
* the Symfony bundle registers `ESP256`, `ESP384`, `ESP512` and `Ed25519`, so they belong to the default verification manager.

Add them to your own manager when you build one by hand:

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Cose\Algorithm\Manager;
use Cose\Algorithm\Signature\ECDSA\ES256;
use Cose\Algorithm\Signature\FullySpecified\ESP256;
use Cose\Algorithm\Signature\RSA\RS256;

$algorithmManager = Manager::create()
    ->add(
        ES256::create(),
        ESP256::create(),
        RS256::create()
    )
;
```
{% endcode %}

{% hint style="info" %}
Keep the legacy identifiers in the `pubKeyCredParams` you send to the browser. Advertising a fully-specified identifier there is `NOT RECOMMENDED` by the specification and older user agents may not understand it.
{% endhint %}

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
