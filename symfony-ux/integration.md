# User Authentication

With the version 5.2 of the bundle, the login process is very similar to the username/password login.

First, your login form needs a username field. This field is not required ([usernameless authentication](../pure-php/advanced-behaviours/authentication-without-username.md)). You can indicate the autocomplete method is `webauthn`; this helps browser understanding the purpose of this field.

Second, a hidden field `assertion` where the authenticator assertion will be placed is required. The name `assertion` can be changed, but shall be the same declared in the next step.

```twig
<form action={{ path('app_login') }} method="post">
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

The Stimulus Controller should be configured to fits on your needs. The route names used below are automatically created by the bundle configuration.

The `requestResultField` parameter corresponds to the selector to the hidden field added above. Please use the corresponding field name.

```twig
<form action={{ path('app_login') }} method="post"
    {{ stimulus_controller('@web-auth/webauthn-stimulus',
        {
             requestOptionsUrl: path('webauthn.controller.request.request.login')
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

When authenticators are available on the device and the browser is aware of them, you can simplify the way the users will sign in. When this feature is enable, the user will see the list of available authenticators when focusing on the username field. By selecting an account in the list will automatically perform the authentication actions. There is a simple option to enable this feature:

```twig
{{ stimulus_controller('@web-auth/webauthn-stimulus',
    {
        useBrowserAutofill: true,
        requestOptionsUrl: path('webauthn.controller.request.request.login'),
        requestResultField: 'input[name="assertion"]'
    }
) }}
```
