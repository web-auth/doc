# Installation

The [Symfony UX Initiative](https://symfony.com/ux) enables high-interaction applications directly from Twig templates without writing custom JavaScript. The WebAuthn Stimulus Controller makes implementing passwordless authentication as simple as adding a few Twig functions to your forms.

## Prerequisites

Before installing the Stimulus Controller, you need:

1. **Symfony UX configured** - Follow the [official Symfony UX documentation](https://symfony.com/doc/current/frontend/ux.html)
2. **WebAuthn Bundle installed** - See [Symfony Bundle Installation](../symfony-bundle/bundle-installation.md)

## Installation

Install the WebAuthn Stimulus Controller via Composer:

```bash
composer require web-auth/webauthn-stimulus
```

This command will automatically:
* Install the PHP package
* Register the Stimulus controller via Symfony Flex
* Configure AssetMapper to import the necessary JavaScript files

{% hint style="info" %}
**No build step required!** The package works with Symfony AssetMapper, so you don't need Node.js, npm, yarn, or any JavaScript build tools. The browser imports the JavaScript files directly.
{% endhint %}

## Verify Installation

Check that the Stimulus controller is properly registered in your `assets/controllers.json`:

{% code title="assets/controllers.json" lineNumbers="true" %}
```json
{
    "controllers": {
        "@web-auth/webauthn-stimulus": {
            "enabled": true
        }
    }
}
```
{% endcode %}

## What's Included

The package provides Stimulus controllers that handle:

* **Registration flows** - Calls `navigator.credentials.create()` for you
* **Authentication flows** - Calls `navigator.credentials.get()` for you
* **Base64 encoding/decoding** - Automatically handles data conversion
* **Error handling** - Gracefully handles common WebAuthn errors
* **Browser autofill** - Supports conditional UI for passkey selection
* **Conditional create** (v5.3.0+) - Enhanced conditional UI support for both registration and authentication
* **PRF extension** - Built-in support for the Pseudo-Random Function extension

{% hint style="info" %}
**Enhanced in v5.3.0:** The Stimulus package now provides three controllers:

* **`@web-auth/webauthn-stimulus`** - Combined controller (legacy, still supported)
* **`@web-auth/webauthn-stimulus/authentication`** - Dedicated authentication controller
* **`@web-auth/webauthn-stimulus/registration`** - Dedicated registration controller

The dedicated controllers offer better separation of concerns, enhanced conditional UI support, and improved error handling. The combined controller remains available for backward compatibility.
{% endhint %}

## Basic Usage

Once installed, you can use the Stimulus controller in your Twig templates with two simple functions:

1. **`stimulus_controller()`** - Attach the controller to your form
2. **`stimulus_action()`** - Trigger WebAuthn operations on button clicks

### Registration Form Example

{% code title="templates/registration/register.html.twig" lineNumbers="true" %}
```twig
<form
    action="{{ path('app_register') }}"
    method="post"
    {{ stimulus_controller('@web-auth/webauthn-stimulus', {
        creationOptionsUrl: path('webauthn.controller.creation.creation.new_user'),
        creationResultField: 'input[name="attestation"]'
    }) }}
>
    <input type="text" name="username" required>
    <input type="hidden" name="attestation">

    <button
        type="submit"
        {{ stimulus_action('@web-auth/webauthn-stimulus', 'register') }}
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
    {{ stimulus_controller('@web-auth/webauthn-stimulus', {
        requestOptionsUrl: path('webauthn.controller.request.request.login'),
        requestResultField: 'input[name="assertion"]'
    }) }}
>
    <input type="text" name="username" autocomplete="username webauthn">
    <input type="hidden" name="assertion">

    <button
        type="submit"
        {{ stimulus_action('@web-auth/webauthn-stimulus', 'signin') }}
    >
        Sign In
    </button>
</form>
```
{% endcode %}

## Controller Options

The Stimulus controller accepts various configuration options:

### Registration Options

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `creationOptionsUrl` | string | Yes | URL endpoint that returns `PublicKeyCredentialCreationOptions` |
| `creationResultField` | string | Yes | CSS selector for hidden field to store attestation response |

### Authentication Options

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `requestOptionsUrl` | string | Yes | URL endpoint that returns `PublicKeyCredentialRequestOptions` |
| `requestResultField` | string | Yes | CSS selector for hidden field to store assertion response |
| `useBrowserAutofill` | boolean | No | Enable conditional UI for browser autofill (default: false) |

## Testing Your Installation

Create a simple test page to verify everything works:

{% code title="templates/test/webauthn.html.twig" lineNumbers="true" %}
```twig
{% extends 'base.html.twig' %}

{% block body %}
    <h1>WebAuthn Test</h1>

    {% if app.user %}
        <p>Logged in as: {{ app.user.userIdentifier }}</p>
        <a href="{{ path('app_logout') }}">Logout</a>
    {% else %}
        <h2>Register</h2>
        <form
            action="{{ path('app_register') }}"
            method="post"
            {{ stimulus_controller('@web-auth/webauthn-stimulus', {
                creationOptionsUrl: path('webauthn.controller.creation.creation.new_user'),
                creationResultField: 'input[name="attestation"]'
            }) }}
        >
            <label>
                Username:
                <input type="text" name="username" required>
            </label>
            <input type="hidden" name="attestation">

            <button
                type="submit"
                {{ stimulus_action('@web-auth/webauthn-stimulus', 'register') }}
            >
                Register with WebAuthn
            </button>
        </form>

        <h2>Login</h2>
        <form
            action="{{ path('app_login') }}"
            method="post"
            {{ stimulus_controller('@web-auth/webauthn-stimulus', {
                requestOptionsUrl: path('webauthn.controller.request.request.login'),
                requestResultField: 'input[name="assertion"]'
            }) }}
        >
            <label>
                Username:
                <input type="text" name="username" autocomplete="username webauthn">
            </label>
            <input type="hidden" name="assertion">

            <button
                type="submit"
                {{ stimulus_action('@web-auth/webauthn-stimulus', 'signin') }}
            >
                Sign In with WebAuthn
            </button>
        </form>
    {% endif %}
{% endblock %}
```
{% endcode %}

## Troubleshooting

### Controller Not Found

If you see "Controller '@web-auth/webauthn-stimulus' not found":

1. Verify `assets/controllers.json` includes the controller with `"enabled": true`
2. Clear Symfony cache: `php bin/console cache:clear`
3. Check that AssetMapper is properly configured in `config/packages/asset_mapper.yaml`

### JavaScript Errors

If you see console errors:

1. Ensure you're using **HTTPS** (required for WebAuthn)
2. Check browser console for specific error messages
3. Verify WebAuthn routes are properly configured in `config/routes/webauthn.yaml`
4. Ensure the controller is loaded by checking the browser's network tab

### Assets Not Loading

If the Stimulus controller doesn't load:

1. Verify your base template includes the AssetMapper importmap:
   ```twig
   {% block javascripts %}
       {{ importmap('app') }}
   {% endblock %}
   ```

2. Check that Stimulus is properly configured in your application

## HTTPS Requirement

WebAuthn **requires HTTPS** to function. For development, use the Symfony CLI for automatic HTTPS:

```bash
symfony server:ca:install
symfony serve
```

For production, ensure HTTPS is properly configured on your web server.

## Next Steps

Now that the Stimulus Controller is installed, proceed to:

1. **[User Registration](user-registration.md)** - Create registration forms
2. **[User Authentication](user-authentication.md)** - Implement login flows
3. **[Additional Authenticators](additional-authenticators.md)** - Allow users to register backup devices

## See Also

* [Symfony UX Documentation](https://symfony.com/doc/current/frontend/ux.html) - Official UX guide
* [Stimulus Documentation](https://stimulus.hotwired.dev/) - Stimulus framework reference
* [WebAuthn Bundle](../symfony-bundle/bundle-installation.md) - Backend configuration
* [JavaScript Integration](../prerequisites/javascript.md) - Manual JavaScript implementation
