# Input Loading

In general, the data you receive is an encoded JSON object. The library provides utilities to convert the string into objects.

To do so, you will need an Attestation Statement Support Manager and a Serializer.

## Attestation Statement Support Manager

Authenticator Responses may contain an Attestation Statement. This attestation holds data regarding the authenticator depending on several factors such as its manufacturer and model, what you asked in the options, the capabilities of the browser or what the user allowed.

### Supported Attestation Statement Types

The following attestation types are supported. Note that you should only use the `none` one unless you have specific needs described in [the dedicated page](../webauthn-in-a-nutshell/attestation-and-metadata-statement.md).

* `none`: no attestation is provided.
* `fido-u2f`: for non-FIDO2 compatible devices (old FIDO / U2F security tokens).
* `packed`: generally used by authenticators with limited resources (e.g. secure elements). It uses a very compact but still extensible encoding method.
* `android key`: commonly used by old or disconnected Android devices.
* ~~`android safety net`: for new Android devices like smartphones~~ (deprecated).
* `trusted platform module`: for devices with built-in security chips.
* `apple`: for Apple devices

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\AttestationStatement\AttestationStatementSupportManager;
use Webauthn\AttestationStatement\NoneAttestationStatementSupport;

// The manager will receive data to load and select the appropriate 
$attestationStatementSupportManager = AttestationStatementSupportManager::create();
$attestationStatementSupportManager->add(NoneAttestationStatementSupport::create());
```
{% endcode %}

## The Serializer

To convert Authenticator Responses from the encoded string to an object, you will need a Serializer. It only needs the Attestation Statement Support Manager created above.

{% hint style="danger" %}
Before 4.8.0, you were asked to create a PublicKeyCredentialLoader object. For 4.8.0, you can use a Symfony Serializer object. This will become the standard way to load data for 5.0.0.
{% endhint %}

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\Denormalizer\WebauthnSerializerFactory;

$factory = new WebauthnSerializerFactory($attestationStatementSupportManager);
$serializer = $factory->create();
```
{% endcode %}

## Loding Data

In general, the data you receive looks like as follows.

{% code lineNumbers="true" %}
```json
{
    "id":"KVb8CnwDjpgAo[…]op61BTLaa0tczXvz4JrQ23usxVHA8QJZi3L9GZLsAtkcVvWObA",
    "type":"public-key",
    "rawId":"KVb8CnwDjpgAo[…]rQ23usxVHA8QJZi3L9GZLsAtkcVvWObA==",
    "response":{
        "clientDataJSON":"eyJjaGFsbGVuZ2UiOiJQbk1hVjBVTS[…]1iUkdHLUc4Y3BDSdGUifQ==",
        "attestationObject":"o2NmbXRmcGFja2VkZ2F0dFN0bXSj[…]YcGhf"
    }
}
```
{% endcode %}

{% hint style="info" %}
Only this type of input is supported. If you receive other forms of data, please contact us.
{% endhint %}

<pre class="language-php" data-line-numbers><code class="lang-php"><strong>&#x3C;?php
</strong>
declare(strict_types=1);

use Webauthn\PublicKeyCredential;

// $data corresponds to the JSON object showed above
$publicKeyCredential = $serializer->deserialize(
    $data,
    PublicKeyCredential::class,
    'json'
);
</code></pre>

If the data is correctly loaded, the variable `$publicKeyCredential` will be an instance of `Webauthn\PublicKeyCredential`. An exception is thrown in case of an error.

{% hint style="danger" %}
At this stage, the data is not verified.
{% endhint %}
