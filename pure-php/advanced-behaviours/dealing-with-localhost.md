# Dealing with “localhost”

## Secured Context

If your are working on a development environment, `https` may not be available but the context could be considered as secured. You can bypass the scheme verification by passing the list of rpIds you consider secured.

{% hint style="danger" %}
Please be careful using this feature. It should NOT be used in production.
{% endhint %}

```php
$publicKeyCredentialSource = $authenticatorAttestationResponseValidator->check(
    $authenticatorAttestationResponse,
    $publicKeyCredentialCreationOptions,
    'my-application.com',
    ['localhost']
);
```

```php
$publicKeyCredentialSource = $authenticatorAssertionResponse->check(
    $publicKeyCredential->getRawId(),
    $authenticatorAssertionResponse,
    $publicKeyCredentialRequestOptions,
    'my-application.com',
    $userHandle,
    ['localhost']
);
```
