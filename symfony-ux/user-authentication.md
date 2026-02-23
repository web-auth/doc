# User Authentication

With the version 5.2 of the bundle, the login process is very similar to the username/password login.

First, your login form needs a username field. This field is not required ([usernameless authentication](../pure-php/advanced-behaviours/authentication-without-username.md)). You can indicate the autocomplete method is `webauthn`; this helps the browser understand the purpose of this field.

Second, a hidden field `assertion` where the authenticator assertion will be placed is required. The name `assertion` can be changed, but shall be the same declared in the next step.

```twig
<form action="{{ path('app_login') }}" method="post">
    <label for="username">Username</label>
    <input name="username" type="text" id="username" placeholder="Type your username here" autocomplete="username webauthn">
    <input type="hidden" id="assertion" name="assertion">

    <button type="submit">
        Sign in
    </button>
</form>
```

You now have only two Twig functions to call: `stimulus_controller` and `stimulus_action`.&#x20;

* The first one is placed on the `form` level;
* The latter on the `button`.

The Stimulus Controller should be configured to fit your needs. The route names used below are automatically created by the bundle configuration.

The `requestResultField` parameter corresponds to the selector to the hidden field added above. Please use the corresponding field name.

```twig
<form action={{ path('app_login') }} method="post"
    {{ stimulus_controller('@web-auth/webauthn-stimulus',
        {
             requestOptionsUrl: path('webauthn.controller.request.request.login'),
             requestResultField: 'input[name="assertion"]'
        }
    ) }}
>
    <label for="username">Username</label>
    <input name="username" type="text" id="username" placeholder="Type your username here" autocomplete="username webauthn">
    <input type="hidden" id="assertion" name="assertion">

    <button
        type="submit"
        {{ stimulus_action('@web-auth/webauthn-stimulus', 'signin') }}
    >
        Sign in
    </button>
</form>

```

### Redirection after login

The behavior after the login is managed by your Symfony Security Authenticator.

### Browser Autofill

When authenticators are available on the device and the browser is aware of them, you can simplify the way users will sign in. When this feature is enabled, the user will see the list of available authenticators when focusing on the username field. Selecting an account in the list will automatically perform the authentication actions. There is a simple option to enable this feature:

```twig
{{ stimulus_controller('@web-auth/webauthn-stimulus',
    {
        useBrowserAutofill: true,
        requestOptionsUrl: path('webauthn.controller.request.request.login'),
        requestResultField: 'input[name="assertion"]'
    }
) }}
```

## Dedicated Authentication Controller

{% hint style="info" %}
**New in v5.3.0:** The Stimulus package now provides dedicated controllers for authentication and registration, in addition to the combined controller.
{% endhint %}

The new `authentication-controller` provides a focused controller specifically for sign-in flows, with built-in support for conditional UI and better error handling.

### Basic Usage

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
        Sign in
    </button>
</form>
```
{% endcode %}

### Conditional UI (Autofill)

The dedicated controller supports conditional UI natively:

{% code lineNumbers="true" %}
```twig
{{ stimulus_controller('@web-auth/webauthn-stimulus/authentication', {
    optionsUrl: path('webauthn.controller.request.request.login'),
    resultUrl: path('app_login'),
    submitViaForm: true,
    conditionalUi: true
}) }}
```
{% endcode %}

### Authentication Controller Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `optionsUrl` | string | `/authentication/options` | URL to fetch assertion options |
| `resultUrl` | string | `/authentication/verify` | URL to submit the assertion response |
| `submitViaForm` | boolean | `false` | Submit via form instead of fetch |
| `successRedirectUri` | string | - | URL to redirect after successful auth |
| `conditionalUi` | boolean | `false` | Enable conditional UI (autofill) |
| `verifyAutofillInput` | boolean | `true` | Verify autofill input fields exist |

### Authentication Controller Targets

| Target | Description |
|--------|-------------|
| `username` | Input field for the username |
| `userVerification` | Input field for user verification preference |
| `result` | Hidden input for the assertion response |
