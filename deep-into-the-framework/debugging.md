# Debugging

If you have troubles during the development of your application or if you want to keep track of every critical/error messages in production, you can use a [PSR-3 compatible logger](https://www.php-fig.org/psr/psr-3/).

The following classes provide a `setLogger` function to pass your PSR-3 logger instance.

* `Webauthn\AttestationStatement\AttestationObjectLoader`
* `Webauthn\AuthenticatorAssertionResponseValidator`
* `Webauthn\AuthenticatorAttestationResponseValidator`
* `Webauthn\PublicKeyCredentialLoader`
* `Webauthn\Counter\ThrowExceptionIfInvalid`
