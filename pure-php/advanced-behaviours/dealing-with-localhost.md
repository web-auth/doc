# Cross Origin Authentication

## Default behavior

By default, the current host is used, and the origin parameter returned in the WebAuthn authenticator response must be identical. Additionally, only the HTTPS scheme is allowed. This may cause issues in several situations, such as:

* **Using multiple origins**: If your application is accessible from multiple domains (e.g., `example.com` and `app.example.com`), the default behavior will reject authentication attempts from different origins.
* **Supporting native applications**: Companion applications on mobile or desktop may use different schemes (e.g., `android:apk-key-hash://` or `ios:bundle-id://`), which are not covered by the default host-based validation.
* **Subdomain authentication**: Some deployments use a wildcard subdomain approach (e.g., `tenant1.example.com`, `tenant2.example.com`), but strict origin checking will reject authentication if the relying party ID does not match exactly.
* **Development and testing environments**: Local development often uses `http://localhost`, which is not allowed by default due to the HTTPS requirement.

## Customizing Allowed Origins

To handle these cases, you can explicitly define a list of allowed origins when configuring the WebAuthn authentication process using the method `setAllowedOrigins`.

{% hint style="info" %}
The method `setSecuredRelyingPartyId` is deprecated since 5.2.0
{% endhint %}

### Allowing Multiple Origins

If your application supports multiple domains, specify them explicitly:

This ensures that authentication requests coming from any of these origins are accepted.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\CeremonyStep\CeremonyStepManagerFactory;

$csmFactory = new CeremonyStepManagerFactory();
$csmFactory->setAllowedOrigins([
    'https://acme.com',
    'https://acme.fr',
    'https://acme.de',
    'https://old-acme.com'
]);
```
{% endcode %}

### Allowing Subdomains

To allow authentication from subdomains dynamically, enable the `allowSubdomains` flag:

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\CeremonyStep\CeremonyStepManagerFactory;

$csmFactory = new CeremonyStepManagerFactory();
$csmFactory->setAllowedOrigins(['https://acme.com'], true);
```
{% endcode %}

With this setting, authentication requests from `https://sub.acme.com` will be considered valid.

{% hint style="danger" %}
Do not use TLD and the sub domain flag together! `$csmFactory->setSecuredRelyingPartyId(['com'], true);`
{% endhint %}

### Supporting Native Applications

For native applications, origins are different from traditional web URLs. If your application integrates with mobile apps, ensure that relevant origins are included:

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\CeremonyStep\CeremonyStepManagerFactory;

$csmFactory = new CeremonyStepManagerFactory();
$csmFactory->setAllowedOrigins([
    'https://acme.com',
    'android:apk-key-hash://your-app-hash',
    'ios:bundle-id://your.bundle.id',
]);
```
{% endcode %}

This configuration allows authentication responses from web browsers and native applications.

### Enabling `http://localhost` for Development

During local development, you may need to allow `http://localhost`. Since WebAuthn generally requires HTTPS, this is an exception for local testing. To allow it:

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\CeremonyStep\CeremonyStepManagerFactory;

$csmFactory = new CeremonyStepManagerFactory();
$csmFactory->setAllowedOrigins([
    'http://localhost.com',
]);
```
{% endcode %}

Be aware that this should only be enabled in non-production environments.

## Security Considerations

When modifying allowed origins, ensure that:

* You **only** allow trusted origins to prevent phishing attacks.
* You enforce HTTPS wherever possible. Some browsers support `https://localhost`, so prefer it over `http://localhost` when feasible.

By properly configuring allowed origins, you can support a variety of WebAuthn use cases while maintaining security best practices.
