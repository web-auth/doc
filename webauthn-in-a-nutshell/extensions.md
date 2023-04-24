# Extensions

The mechanism for generating public key credentials, as well as requesting and generating Authentication assertions, can be extended to suit particular use cases. Each case is addressed by defining a registration extension.

Standard extensions are usually listed in the dedicated IANA Registry available at [https://www.iana.org/assignments/webauthn/webauthn.xhtml](https://www.iana.org/assignments/webauthn/webauthn.xhtml)

{% hint style="warning" %}
This library is ready to handle extension inputs and outputs, but no concrete implementations are provided, except for the `appip` extension.

It is up to you, depending on the extensions you want to support, to create the extension handlers.
{% endhint %}
