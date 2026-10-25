# Fake Credentials

In order to prevent username enumeration, random credentials are set when a username is passed but no user entity is found.

A very simple service is provided. If you want to change the way the fake credentials are generated, you can create a custom service. The service shall implement the `Webauthn\FakeCredentialGenerator` interface.

```php title="src/CustomCredentialGenerator.php" linenums="1"
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
        // Note that for a given username you should always return the same credentials.
    }
}
```

Then, declare this service in the container and use it in your bundle configuration.

```yaml title="config/packages/webauthn.yaml" linenums="1"
webauthn:
    fake_credential_generator: App\CustomCredentialGenerator
```
