# Authentication without username

With Webauthn, it is possible to authenticate a user without username. This behavior implies several constraints:

1. During the registration of the authenticator, a [Resident Key must have been asked](authenticator-selection-criteria.md#resident-key),
2. The user verification is required,
3. The list of allowed authenticators must be empty

{% hint style="info" %}
In case of failure, you should continue with the standard authentication process i.e. by asking the username of the user.
{% endhint %}

### Examples

#### The Easy Way

Selection criterias for the registration of the authenticator:

```php
use Webauthn\AuthenticatorSelectionCriteria;
use Webauthn\PublicKeyCredentialCreationOptions;

$authenticatorSelectionCriteria = new AuthenticatorSelectionCriteria(
    AuthenticatorSelectionCriteria::AUTHENTICATOR_ATTACHMENT_NO_PREFERENCE,
    true,                                                                  // Resident key required
    AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_REQUIRED // User verification required
);
```

The Request Options:

```php
<?php

use Webauthn\PublicKeyCredentialRequestOptions;

$ublicKeyCredentialRequestOptions = $server->generatePublicKeyCredentialRequestOptions(
    PublicKeyCredentialRequestOptions::USER_VERIFICATION_REQUIREMENT_REQUIRED,
);
```

