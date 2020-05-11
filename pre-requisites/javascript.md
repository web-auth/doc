---
description: Examples for dynamic interactions
---

# Javascript

You will interact with the authenticators through an HTML page and Javascript using the Webauthn API.

A package is available at [https://github.com/web-auth/webauthn-helper](https://github.com/web-auth/webauthn-helper). It contains functions that will ease the interaction with the login or the registration endpoints.

{% hint style="danger" %}
It is mandatory to use the HTTPS scheme to use Webauthn otherwise it will not work.
{% endhint %}

## Installation

You can use npm or yarn to install the package:

```bash
npm i webauthn-helper
#or
yarn add webauthn-helper
```

## Registration

```javascript
// Import the registration hook
import {useRegistration} from 'webauthn-helper';

// Create your register function.
// By default the urls are "/register" and "/register/options"
// but you can change those urls if needed.
const register = useRegistration({
    actionUrl: '/api/register',
    optionsUrl: '/api/register/options'
});


// We can call this register function whenever we need (e.g. form submission)
register({
    username: 'john.doe',
    displayName: 'JD'
})
    .then((response) => console.log('Registration success'))
    .catch((error) => console.log('Registration failure'))
;
```

Additional options can be set during the registration process. See the section “Deep into the framework” to know more. Hereafter another example:

```javascript
register({
    username: 'john.doe',
    displayName: 'JD',
    attestation: 'none',
    authenticatorSelection: {
        authenticatorAttachment: 'platform',
        requireResidentKey: true,
        userVerification: 'required'
    }
})
    .then((response) => console.log('Registration success'))
    .catch((error) => console.log('Registration failure'))
;
```

## Authentication

```javascript
// Import the registration hook
import {useLogin} from 'webauthn-helper';

// Create your login function.
// By default the urls are "/login" and "/login/options"
// but you can change those urls if needed.
const login = useLogin({
    actionUrl: '/api/login',
    optionsUrl: '/api/login/options'
});


// We can call this login function whenever we need (e.g. form submission)
register({
    username: 'john.doe'
})
    .then((response) => console.log('Registration success'))
    .catch((error) => console.log('Registration failure'))
;
```

As done during the registration, additional options are available. See the section “Deep into the framework” to know more. Hereafter another example:

```javascript
register({
    username: 'john.doe',
    userVerification: 'required'
})
    .then((response) => console.log('Registration success'))
    .catch((error) => console.log('Registration failure'))
;
```

