# Debugging

If you have troubles during the development of your application or if you want to keep track of every critical/error messages in production, you can use a [PSR-3 compatible logger](https://www.php-fig.org/psr/psr-3/).

## The Hard Way

Prior to version 3.3, the following classes have an optional constructor parameter `$logger` that can accept the logging service. From version 3.3 onwards you should use their `setLogger` function instead.

* `Webauthn\AttestationStatement\AttestationObjectLoader`
* `Webauthn\AuthenticatorAssertionResponseValidator`
* `Webauthn\AuthenticatorAttestationResponseValidator`
* `Webauthn\PublicKeyCredentialLoader`
* `Webauthn\Counter\ThrowExceptionIfInvalid`

## The Symfony Way

{% code title="config/packages/webauthn.yaml" %}
```yaml
webauthn:
    logger: App\Service\MyPsr3Logger
```
{% endcode %}
