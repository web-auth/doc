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

{% hint style="info" %}
**Changed in v5.3.3:** an entry that is not a URL, i.e. that has a scheme but no host, such as `android:apk-key-hash:<base64>`, is stored as is and compared verbatim with the `origin` field of the client data. Entries made of a host only, such as `acme.com`, are still normalized to `https://acme.com` because WebAuthn requires TLS.

Copy the facet identifier exactly as the application sends it. Before v5.3.3 these entries went through the host normalization and were turned into unusable origins such as `https://android:apk-key-hash:...`, so they never matched.
{% endhint %}

### Enabling `http://localhost` for Development

During local development, you may need to allow `http://localhost`. Since WebAuthn generally requires HTTPS, this is an exception for local testing. To allow it:

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\CeremonyStep\CeremonyStepManagerFactory;

$csmFactory = new CeremonyStepManagerFactory();
$csmFactory->setAllowedOrigins([
    'http://localhost',
]);
```
{% endcode %}

Be aware that this should only be enabled in non-production environments.

## Top Origin Validation (Cross-Origin iframes)

When the ceremony takes place in a cross-origin iframe, for example a page of `auth.example.com` embedded in `app.example.com`, the browser sets `crossOrigin` to `true` in the client data and adds the origin of the embedding page in a `topOrigin` field.

{% hint style="danger" %}
**Changed in v5.3.4:** a response that carries a `topOrigin` is rejected unless a `TopOriginValidator` is configured. Previous versions accepted such a response without checking anything, which means any site embedding your ceremony in an iframe was accepted.

Applications that do not use iframes have nothing to do: a same-origin ceremony has no `topOrigin` at all. Applications that are legitimately embedded must opt in with one of the validators below, otherwise their ceremonies now fail.
{% endhint %}

The step also rejects a response whose client data carries a `topOrigin` while `crossOrigin` is `false`, as the two fields contradict each other.

### Using the Built-in Host Validator

`HostTopOriginValidator` compares the received top origin with a fixed value:

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\CeremonyStep\CeremonyStepManagerFactory;
use Webauthn\CeremonyStep\HostTopOriginValidator;

$csmFactory = new CeremonyStepManagerFactory();
$csmFactory->enableTopOriginValidator(
    new HostTopOriginValidator('https://app.example.com')
);
```
{% endcode %}

{% hint style="warning" %}
The comparison is strict. The expected value is the top origin as the browser sends it, i.e. a serialized origin such as `https://app.example.com`, and not a bare host name.
{% endhint %}

### Using a Custom Validator

When several parent origins are allowed, implement the `TopOriginValidator` interface:

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\CeremonyStep\TopOriginValidator;
use Webauthn\Exception\AuthenticatorResponseVerificationException;

final readonly class AllowedTopOriginsValidator implements TopOriginValidator
{
    /**
     * @param string[] $allowedTopOrigins
     */
    public function __construct(
        private array $allowedTopOrigins
    ) {
    }

    public function validate(string $topOrigin): void
    {
        if (!in_array($topOrigin, $this->allowedTopOrigins, true)) {
            throw AuthenticatorResponseVerificationException::create(
                'The top origin is not allowed.'
            );
        }
    }
}
```
{% endcode %}

Then register it:

{% code lineNumbers="true" %}
```php
$csmFactory->enableTopOriginValidator(
    new AllowedTopOriginsValidator([
        'https://app.example.com',
        'https://dashboard.example.com',
    ])
);
```
{% endcode %}

A validator set on the factory applies to both the creation and the request ceremonies. `disableTopOriginValidator()` removes it, which is useful when the same factory is reused to build a manager for a context where iframes are not allowed.

## Security Considerations

When modifying allowed origins, ensure that:

* You **only** allow trusted origins to prevent phishing attacks.
* You enforce HTTPS wherever possible. Some browsers support `https://localhost`, so prefer it over `http://localhost` when feasible.
* You keep the list of allowed top origins as small as possible. Any site listed there is allowed to embed your ceremony.

By properly configuring allowed origins, you can support a variety of WebAuthn use cases while maintaining security best practices.
