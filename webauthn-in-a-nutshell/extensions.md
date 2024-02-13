# Extensions

The mechanism for generating public key credentials, as well as requesting and generating Authentication assertions, can be extended to suit particular use cases. Each case is addressed by defining a registration extension.

Standard extensions are usually listed in the dedicated IANA Registry available at [https://www.iana.org/assignments/webauthn/webauthn.xhtml](https://www.iana.org/assignments/webauthn/webauthn.xhtml)

Among the available extensions, you have:

* `loc`: The location registration extension and authentication extension provides the client device's current location to the WebAuthn Relying Party, if supported by the client platform and subject to user consent.
* `hmac-secret`: This registration extension and authentication extension enables the platform to retrieve a symmetric secret scoped to the credential from the authenticator.
* `minPinLength`: This registration extension returns the current minimum PIN length value to the Relying Party.

{% hint style="warning" %}
This library is ready to handle extension inputs and outputs, but no concrete implementations are provided.

It is up to you, depending on the extensions you want to support, to create the extension handlers.
{% endhint %}
