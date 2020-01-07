# The Relaying Party

The Relaying Party \(or `rp`\) corresponds to the application that will ask for the user to interact with the authenticator.

The library provides a simple class to handle the rp information: `Webauthn\PublicKeyCredentialRpEntity`.

```php
<?php

use Webauthn\PublicKeyCredentialRpEntity;

$rpEntity = new PublicKeyCredentialRpEntity(
    'ACME Webauthn Server', // The application name
    'acme.com'              // The application ID = the domain
);
```

This `$rpEntity` object will be useful for the next steps.

{% hint style="success" %}
The ID can be `null`, the domain or sub-domain of your application.
{% endhint %}

{% hint style="info" %}
Even if it is optional, we highly recommend to set the application ID. If absent, the current domain will be used
{% endhint %}

{% hint style="warning" %}
 The scheme, userinfo, port, path, user… are not allowed.

Allowed: `www.sub.domain.com`, `sub.domain.com`, `domain.com` 

Not allowed:`com`, `www.sub.domain.com:1337`, `https://domain.com:443`, `sub.domain.com/index`, `https://user:password@www.domain.com`.
{% endhint %}

Your application may also have a logo. You can indicate this logo as third argument. Please note that for safety reason this icon is a priori authenticated URL i.e. an image that uses the `data` scheme.

```php
<?php

use Webauthn\PublicKeyCredentialRpEntity;

$rpEntity = new PublicKeyCredentialRpEntity(
    'ACME Webauthn Server',
    'acme.com',
    'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABQAAAAUCAMAAAC6V+0/AAAAwFBMVEXm7NK41k3w8fDv7+q01Tyy0zqv0DeqyjOszDWnxjClxC6iwCu11z6y1DvA2WbY4rCAmSXO3JZDTxOiwC3q7tyryzTs7uSqyi6tzTCmxSukwi9aaxkWGga+3FLv8Ozh6MTT36MrMwywyVBziSC01TbT5ZW9z3Xi6Mq2y2Xu8Oioxy7f572qxzvI33Tb6KvR35ilwTmvykiwzzvV36/G2IPw8O++02+btyepyDKvzzifvSmw0TmtzTbw8PAAAADx8fEC59dUAAAA50lEQVQYV13RaXPCIBAG4FiVqlhyX5o23vfVqUq6mvD//1XZJY5T9xPzzLuwgKXKslQvZSG+6UXgCnFePtBE7e/ivXP/nRvUUl7UqNclvO3rpLqofPDAD8xiu2pOntjamqRy/RqZxs81oeVzwpCwfyA8A+8mLKFku9XfI0YnSKXnSYZ7ahSII+AwrqoMmEFKriAeVrqGM4O4Z+ADZIhjg3R6LtMpWuW0ERs5zunKVHdnnnMLNQqaUS0kyKkjE1aE98b8y9x9JYHH8aZXFMKO6JFMEvhucj3Wj0kY2D92HlHbE/9Vk77mD6srRZqmVEAZAAAAAElFTkSuQmCC'
);
```

{% hint style="info" %}
The Webauthn specification does not set any limit for the length of the third argument.
{% endhint %}

{% hint style="warning" %}
The icon may be ignored by browsers, especially if its length is greater than 128 bytes.
{% endhint %}

