# Installation

The [Symfony UX Initiative](https://symfony.com/ux) enables high-interaction applications directly from Twig templates without writing custom JavaScript. The WebAuthn Stimulus Controller makes implementing passwordless authentication as simple as adding a few Twig functions to your forms.

## Prerequisites

Before installing the Stimulus Controller, you need:

1. **Symfony UX configured** — follow the [official Symfony UX documentation](https://symfony.com/doc/current/frontend/ux.html)
2. **WebAuthn Bundle installed** — see [Symfony Bundle Installation](../symfony-bundle/bundle-installation.md)

## Installation

Since v5.3.0 the Stimulus controllers are published on npm as [`@web-auth/webauthn-stimulus`](https://www.npmjs.com/package/@web-auth/webauthn-stimulus). The npm package is the canonical and only maintained source going forward — the legacy `web-auth/webauthn-stimulus` Composer package is deprecated and will be removed in 6.0.0.

{% hint style="danger" %}
**Do not declare this package in `assets/controllers.json`.** That file is reserved for Symfony **PHP** UX packages. `Symfony\UX\StimulusBundle` resolves every entry against an installed Composer package, so adding `@web-auth/webauthn-stimulus` there throws `Could not find package "web-auth/webauthn-stimulus" referred to from controllers.json` as soon as the Composer wrapper is gone (the very thing we want). Register the controllers from your JavaScript code instead — see below.
{% endhint %}

### 1. Pin the package in your importmap

With Symfony AssetMapper:

```bash
php bin/console importmap:require @web-auth/webauthn-stimulus
```

With Webpack Encore, Vite, esbuild or any other bundler:

```bash
npm install @web-auth/webauthn-stimulus
```

The package depends on `@hotwired/stimulus` and `@simplewebauthn/browser`, which are pulled in automatically by AssetMapper or your bundler.

### 2. Register the controllers in your Stimulus app

Import the controllers from the package entry point and register them on your Stimulus `Application`. Use the same `--`-separated identifiers that `stimulus_controller('@web-auth/webauthn-stimulus/...')` produces, so the existing Twig helpers keep working unchanged:

{% code title="assets/bootstrap.js" lineNumbers="true" %}
```javascript
import { Application } from '@hotwired/stimulus';
import {
    AuthenticationController,
    RegistrationController,
    WebauthnController,
} from '@web-auth/webauthn-stimulus';

const app = Application.start();

// Dedicated controllers (recommended since v5.3.0)
app.register('web-auth--webauthn-stimulus--authentication', AuthenticationController);
app.register('web-auth--webauthn-stimulus--registration', RegistrationController);

// Legacy combined controller — only register if you still rely on it
app.register('web-auth--webauthn-stimulus', WebauthnController);
```
{% endcode %}

{% hint style="info" %}
**Why these identifiers?** `stimulus_controller('@vendor/pkg/ctrl')` simply transforms the name to a flat string by stripping the leading `@` and turning `/` into `--`. So `stimulus_controller('@web-auth/webauthn-stimulus/authentication')` renders `data-controller="web-auth--webauthn-stimulus--authentication"` — the same identifier you registered in JS. Existing Twig templates that use the `@`-prefixed form keep working without changes.

The exact location of your Stimulus bootstrap file depends on your project layout. With the default Symfony AssetMapper recipe it is `assets/bootstrap.js` or `assets/app.js`. Anywhere your `Application.start()` runs is fine.
{% endhint %}

### Deprecated: install via Composer (`web-auth/webauthn-stimulus`)

{% hint style="danger" %}
This installation path is kept for backward compatibility only. New projects should use the npm package above.
{% endhint %}

```bash
composer require web-auth/webauthn-stimulus
```

This command installs the PHP wrapper, which used to register the Stimulus controllers via Symfony Flex and configure AssetMapper to import them from a vendored copy. As of v5.3.0 the same JavaScript ships on npm and the PHP wrapper no longer brings any added value — it is scheduled for removal in 6.0.0.

## What's Included

The package provides Stimulus controllers that handle:

* **Registration flows** — calls `navigator.credentials.create()` for you
* **Authentication flows** — calls `navigator.credentials.get()` for you
* **Base64 encoding/decoding** — automatically handles data conversion
* **Error handling** — gracefully handles common WebAuthn errors
* **Browser autofill** — supports conditional UI for passkey selection
* **Conditional create** (v5.3.0+) — enhanced conditional UI support for both registration and authentication
* **PRF extension** — built-in support for the Pseudo-Random Function extension

{% hint style="info" %}
**Three controllers ship in the package:**

* `WebauthnController` — combined controller (legacy, still supported)
* `AuthenticationController` — dedicated authentication controller
* `RegistrationController` — dedicated registration controller

The dedicated controllers offer better separation of concerns, enhanced conditional UI support, and improved error handling. The combined controller remains available for backward compatibility.
{% endhint %}

## Basic Usage

Once the controllers are registered, you can use them in your Twig templates with the standard Symfony UX helpers:

1. **`stimulus_controller()`** — attaches a controller to your form
2. **`stimulus_action()`** — triggers WebAuthn operations on button clicks
3. **`stimulus_target()`** — marks an input as a controller target

The first argument is the package-prefixed name — it will resolve to the `--`-separated identifier you registered in step 2.

### Registration Form Example

{% code title="templates/registration/register.html.twig" lineNumbers="true" %}
```twig
<form
    action="{{ path('app_register') }}"
    method="post"
    {{ stimulus_controller('@web-auth/webauthn-stimulus/registration', {
        optionsUrl: path('webauthn.controller.creation.request.new_user'),
        resultUrl: path('app_register'),
        submitViaForm: true
    }) }}
>
    <input
        type="text"
        name="username"
        required
        {{ stimulus_target('@web-auth/webauthn-stimulus/registration', 'username') }}
    >
    <input
        type="hidden"
        name="attestation"
        {{ stimulus_target('@web-auth/webauthn-stimulus/registration', 'result') }}
    >

    <button
        type="button"
        {{ stimulus_action('@web-auth/webauthn-stimulus/registration', 'register') }}
    >
        Register
    </button>
</form>
```
{% endcode %}

### Authentication Form Example

{% code title="templates/security/login.html.twig" lineNumbers="true" %}
```twig
<form
    action="{{ path('app_login') }}"
    method="post"
    {{ stimulus_controller('@web-auth/webauthn-stimulus/authentication', {
        optionsUrl: path('webauthn.controller.request.request.login'),
        resultUrl: path('app_login'),
        submitViaForm: true
    }) }}
>
    <input
        type="text"
        name="username"
        autocomplete="username webauthn"
        {{ stimulus_target('@web-auth/webauthn-stimulus/authentication', 'username') }}
    >
    <input
        type="hidden"
        name="assertion"
        {{ stimulus_target('@web-auth/webauthn-stimulus/authentication', 'result') }}
    >

    <button
        type="button"
        {{ stimulus_action('@web-auth/webauthn-stimulus/authentication', 'authenticate') }}
    >
        Sign In
    </button>
</form>
```
{% endcode %}

## Controller Options

The dedicated controllers accept the following options. See the [User Authentication](user-authentication.md) and [User Registration](user-registration.md) pages for the full list of values, targets, actions and events.

### Registration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `optionsUrl` | string | `/registration/options` | URL to fetch creation options |
| `resultUrl` | string | `/registration/verify` | URL to submit the attestation response |
| `submitViaForm` | boolean | `false` | Submit via form instead of fetch |
| `successRedirectUri` | string | – | URL to redirect after successful registration |
| `autoRegister` | boolean | `false` | Auto-start registration on page load |

### Authentication Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `optionsUrl` | string | `/authentication/options` | URL to fetch assertion options |
| `resultUrl` | string | `/authentication/verify` | URL to submit the assertion response |
| `submitViaForm` | boolean | `false` | Submit via form instead of fetch |
| `successRedirectUri` | string | – | URL to redirect after successful auth |
| `conditionalUi` | boolean | `false` | Enable conditional UI (autofill) |

## Troubleshooting

### `Could not find package "web-auth/webauthn-stimulus" referred to from controllers.json`

You added `@web-auth/webauthn-stimulus` to `assets/controllers.json`. **Remove that entry.** The npm package is not a Symfony UX PHP package — `controllers.json` is wrong here. Register the controllers from your JS bootstrap file instead, as shown in [step 2 above](#2-register-the-controllers-in-your-stimulus-app).

### Controller not found

If you see *"application Error: Controller 'web-auth--webauthn-stimulus--authentication' is not registered"* (or similar):

1. Verify the identifier in `app.register('web-auth--webauthn-stimulus--authentication', ...)` matches the rendered `data-controller` attribute. Remember `stimulus_controller('@web-auth/webauthn-stimulus/authentication')` produces `data-controller="web-auth--webauthn-stimulus--authentication"`.
2. Make sure your bootstrap file is included by your importmap (`{{ importmap('app') }}` in your base template).
3. Clear Symfony cache: `php bin/console cache:clear`.

### JavaScript errors

If you see console errors:

1. Ensure you're using **HTTPS** (required for WebAuthn).
2. Check browser console for specific error messages.
3. Verify WebAuthn routes are properly configured in `config/routes/webauthn.yaml`.
4. Ensure the controller is loaded by checking the browser's network tab.

## HTTPS Requirement

WebAuthn **requires HTTPS** to function. For development, use the Symfony CLI for automatic HTTPS:

```bash
symfony server:ca:install
symfony serve
```

For production, ensure HTTPS is properly configured on your web server.

## Next Steps

Now that the Stimulus Controller is installed, proceed to:

1. **[User Registration](user-registration.md)** — create registration forms
2. **[User Authentication](user-authentication.md)** — implement login flows
3. **[Additional Authenticators](additional-authenticators.md)** — allow users to register backup devices

## See Also

* [Symfony UX Documentation](https://symfony.com/doc/current/frontend/ux.html) — official UX guide
* [Stimulus Documentation](https://stimulus.hotwired.dev/) — Stimulus framework reference
* [WebAuthn Bundle](../symfony-bundle/bundle-installation.md) — backend configuration
* [JavaScript Integration](../prerequisites/javascript.md) — manual JavaScript implementation
