---
description: Runnable applications to see the library at work
---

# Demos

Every demo lives in a dedicated repository: [web-auth/webauthn-demos](https://github.com/web-auth/webauthn-demos). It hosts seven standalone pure PHP projects and a complete Symfony application, each with its own README.

```bash
git clone https://github.com/web-auth/webauthn-demos.git
cd webauthn-demos
```

## Pure PHP demos

They use `web-auth/webauthn-lib` alone: no framework, no database, no build step. Each one stores its state in a local file and boots with a single command.

| Demo | What it shows |
| --- | --- |
| [basic-demo](https://github.com/web-auth/webauthn-demos/tree/main/basic-demo) | Username based registration, sign in and credential management |
| [usernameless-demo](https://github.com/web-auth/webauthn-demos/tree/main/usernameless-demo) | Sign in without a username, with discoverable credentials and Conditional UI |
| [passkey-upgrade-demo](https://github.com/web-auth/webauthn-demos/tree/main/passkey-upgrade-demo) | Silent enrollment after a password login, with Conditional Create |
| [signal-api-demo](https://github.com/web-auth/webauthn-demos/tree/main/signal-api-demo) | The three Signal methods that keep the authenticator in sync with the server |
| [extensions-demo](https://github.com/web-auth/webauthn-demos/tree/main/extensions-demo) | `credProps`, `credProtect`, `credBlob` and `minPinLength` in a single ceremony |
| [prf-demo](https://github.com/web-auth/webauthn-demos/tree/main/prf-demo) | The PRF extension used for client side encryption, including an offline vault |
| [spc-demo](https://github.com/web-auth/webauthn-demos/tree/main/spc-demo) | Secure Payment Confirmation, in a same-origin, two-origin and three-tier flavour |

Run one of them from its own directory:

{% code lineNumbers="true" %}
```bash
cd basic-demo
./same-origin/run.sh
```
{% endcode %}

The script installs the dependencies on first run and serves the demo on [http://localhost:8000](http://localhost:8000), which the browsers treat as a secure context. Set the `PORT` variable to serve it elsewhere.

## Running them all at once

{% code lineNumbers="true" %}
```bash
./run-all.sh
```
{% endcode %}

Every demo then gets its own port, from 8101 to 8110, so they can be compared side by side. They share the `localhost` Relying Party ID, which means a passkey registered in one of them is offered by the others.

## Symfony application

[symfony-demo](https://github.com/web-auth/webauthn-demos/tree/main/symfony-demo) is a complete application built on the bundle, with FrankenPHP, AssetMapper and Tailwind. It targets Symfony 8.1 and PHP 8.4, and shows the firewall, the Doctrine entities, a custom options storage and the Metadata Statement services in one place.

{% hint style="info" %}
The pure PHP demos exercise features that are not released yet, so they require `web-auth/webauthn-framework` in its `5.4.x-dev` version. The Symfony application targets the released `^5.3`.
{% endhint %}
