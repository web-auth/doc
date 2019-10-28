# Token Binding

Browsers may support the Token Binding protocol \(see [RFC 8471](https://tools.ietf.org/html/rfc8471)\). This protocol defines a way to bind a token \(the Responses in the Webauthn context\) to the underlying TLS layer.

When receiving a Webauthn Response, the property tokenBinding in the CollectedClientData object have one of the following value:

* `null`: the token binding is not supported by the browser
* `"supported"`: the browser supports token binding, but no negociation was performed during the communication
* `"present"`: the browser supports token binding and is present in the response. The token binding ID is provided.

This feature is not yet implemented in the library, but you can decide how the library will react in case of the presence of the token binding ID.

The library provides two concrete classes for the moment:

* `Webauthn\TokenBinding\IgnoreTokenBindingHandler`: the library will ignore the token binding,
* `Webauthn\TokenBinding\TokenBindingNotSupportedHandler`: the library will throw an exception if the token binding is present.

You can change this behavior by creating your own implementation. The handler must implement the interface `Webauthn\TokenBinding\TokenBindingHandler`.

### The Easy Way

{% hint style="info" %}
To Be Written
{% endhint %}

### The Hard Way

{% hint style="info" %}
To Be Written
{% endhint %}

### The Symfony Way

{% hint style="info" %}
To Be Written
{% endhint %}



