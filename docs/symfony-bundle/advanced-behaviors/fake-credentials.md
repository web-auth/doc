# Fake Credentials

In order to prevent username enumeration, random credentials are set when a username is passed but no user entity is found.

A very simple service is provided: `Webauthn\SimpleFakeCredentialGenerator`. It derives the list from the username and from a secret, so that a given username always gets the same credentials while an outsider cannot compute them.

{% hint style="danger" %}
**Security fix in v5.3.5** ([GHSA-gq4g-fpc9-vjfq](https://github.com/web-auth/webauthn-framework/security/advisories/GHSA-gq4g-fpc9-vjfq))

With an empty secret, the generated credentials depend on the username alone and become predictable: an unauthenticated requester can tell a fake response from a real one, which defeats the protection against username enumeration. Building the generator without a secret is deprecated since 5.3.5 and a non-empty secret is required in 6.0.0.

The bundle injects `%kernel.secret%`, so applications using the default service are not affected. Only a manual instantiation of the generator, in a pure PHP application or in a custom service definition, needs to be fixed.
{% endhint %}

{% code lineNumbers="true" %}
```php
<?php

use Webauthn\SimpleFakeCredentialGenerator;

$generator = new SimpleFakeCredentialGenerator(
    $cacheItemPool, // Optional PSR-6 cache pool
    $applicationSecret
);
```
{% endcode %}

If you want to change the way the fake credentials are generated, you can create a custom service. The service shall implement the `Webauthn\FakeCredentialGenerator` interface.

{% code title="src/CustomCredentialGenerator.php" lineNumbers="true" %}
```php
<?php

namespace App;

use Webauthn\FakeCredentialGenerator;
use Webauthn\PublicKeyCredentialDescriptor;

final readonly class CustomCredentialGenerator implements FakeCredentialGenerator
{
    /**
     * @return PublicKeyCredentialDescriptor[]
     */
    public function generate(Request $request, string $username): array
    {
        // Generate your list of fake credentials.
        // For a given username you should always return the same credentials,
        // and an outsider should not be able to compute them: mix a secret
        // of your own into the generation.
    }
}
```
{% endcode %}

Then, declare this service in the container and use it in your bundle configuration.

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    fake_credential_generator: App\CustomCredentialGenerator
```
{% endcode %}
