---
description: Registration and Authentication process overview
---

# Ceremonies

In the Webauthn context, there are two ceremonies:

* The attestation ceremony: it corresponds to the registration of a new authenticator,
* The assertion ceremony: it is used for the authentication of a user.

For both ceremonies, there are two steps to perform:

1. The creation of options: these options are sent to the authenticator and indicate what to do and how.
2. The response of the authenticator: after the user interacted with the authenticator, the authenticator computes a response that has to be verified.

Depending on the options and the capabilities of the authenticator, the user interaction may differ. It can be a simple touch on a button or a complete authentication using biometric means (PIN code, fingerprint, facial recognition…).

![The attestation ceremony](<../.gitbook/assets/registration (1).png>)

![The assertion ceremony](../.gitbook/assets/login.png)
