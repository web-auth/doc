# Dealing with “localhost”

## Secured Context

If your are working on a development environment, `https` may not be available but the context could be considered as secured. You can bypass the scheme verification by passing the list of rpIds you consider secured.

{% hint style="danger" %}
Please be careful using this feature. It should NOT be used in production.
{% endhint %}

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\CeremonyStep\CeremonyStepManagerFactory;

$csmFactory = new CeremonyStepManagerFactory();
$csmFactory->setSecuredRelyingPartyId(['secure.localhost']);
```
{% endcode %}
