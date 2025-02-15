# Javascript

You will interact with the authenticators through an HTML page and Javascript using the Webauthn API.

We highly recommend the use of [@simplewebauthn/browser](https://simplewebauthn.dev/docs/packages/browser/). This library provides lots of easy and useful features and it fully compliant with the specification.

If you use the Symfony UX, you may be interested in the [Stimulus Controller](../symfony-ux/installation.md).

{% hint style="danger" %}
Note that is mandatory to use the HTTPS scheme to use Webauthn otherwise it will not work. This is also mandatory for `localhost`.
{% endhint %}

