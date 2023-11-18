# Integration

Consider the following login form.

```twig
<form>
    <label for="username">Username</label>
    <input name="username" type="text" id="username" placeholder="Type your username here" autocomplete="username">
    
    <label for="password">Password</label>
    <input name="password" type="password" id="password" placeholder="Type your password here" autocomplete="password">
    
    <button type="submit">
        Sign in
    </button>
</form>
```

First step is to remove the password field that is no longer needed. In addition, we can indicate the autocomplete method is `webauthn`; this helps browser understanding the purpose of this field.

```twig
<form>
    <label for="username">Username</label>
    <input name="username" type="text" id="username" placeholder="Type your username here" autocomplete="username webauthn">

    <button type="submit">
        Sign in
    </button>
</form>
```

We now have only two Twig functions to call: `stimulus_controller` and `stimulus_action`. The first one is placed on the `form` level; the latter on the `button`.

```twig
<form
    {{ stimulus_controller('@web-auth/webauthn-stimulus') }}
>
    <label for="username">Username</label>
    <input name="username" type="text" id="username" placeholder="Type your username here" autocomplete="username webauthn">

    <button
        type="submit"
        {{ stimulus_action('@web-auth/webauthn-stimulus', 'signin') }}
    >
        Sign in
    </button>
</form>
```

The Stimulus Controller should be configured to fits on your needs. In particular, the routes to the options and authenticator result.

```twig
{{
    stimulus_controller(
        '@web-auth/webauthn-stimulus/webauthn',
        {
            requestResultUrl: path('webauthn.controller.security.main.request.result'),
            requestOptionsUrl: path('webauthn.controller.security.main.request.options')
        }
    )
}}
```
