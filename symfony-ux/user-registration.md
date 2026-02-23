# User Registration

The Symfony UX WebAuthn package provides Stimulus controllers to simplify the registration of new authenticators. This page explains how to register a new user with WebAuthn or add authenticators to an existing user account.

## New User Registration

When registering a new user, you need to collect basic information (username, display name) and then register their first authenticator.

### Registration Form

Your registration form needs:
1. A **username field** for the user's identifier
2. A **display name field** (optional, can default to username)
3. A **hidden field** for the attestation response

{% code title="templates/registration/register.html.twig" lineNumbers="true" %}
```twig
<form
    action="{{ path('app_register') }}"
    method="post"
    {{ stimulus_controller('@web-auth/webauthn-stimulus',
        {
            creationOptionsUrl: path('webauthn.controller.creation.creation.new_user'),
            creationResultField: 'input[name="attestation"]'
        }
    ) }}
>
    <label for="username">Username</label>
    <input
        type="text"
        id="username"
        name="username"
        required
        autocomplete="username webauthn"
    >

    <label for="displayName">Display Name</label>
    <input
        type="text"
        id="displayName"
        name="displayName"
        placeholder="John Doe"
    >

    <input type="hidden" id="attestation" name="attestation">

    <button
        type="submit"
        {{ stimulus_action('@web-auth/webauthn-stimulus', 'register') }}
    >
        Register with Passkey
    </button>
</form>
```
{% endcode %}

### Bundle Configuration

Configure the bundle to handle new user registration:

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    credential_repository: 'App\Repository\WebauthnCredentialRepository'
    user_repository: 'App\Repository\UserRepository'

    controllers:
        enabled: true
        creation:
            new_user:
                options_path: '/register/webauthn/options'
                result_path: '/register/webauthn'
                user_entity_guesser: 'App\Guesser\NewUserEntityGuesser'
```
{% endcode %}

### User Entity Guesser

Create a guesser to handle new user registration:

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

namespace App\Guesser;

use Symfony\Component\HttpFoundation\Request;
use Webauthn\Bundle\Security\Guesser\UserEntityGuesser;
use Webauthn\PublicKeyCredentialUserEntity;

final class NewUserEntityGuesser implements UserEntityGuesser
{
    public function findUserEntity(Request $request): PublicKeyCredentialUserEntity
    {
        $username = $request->request->get('username');
        $displayName = $request->request->get('displayName', $username);

        // Generate a unique user ID (64 bytes recommended)
        $userHandle = random_bytes(64);

        return PublicKeyCredentialUserEntity::create(
            $username,
            $userHandle,
            $displayName
        );
    }
}
```
{% endcode %}

{% hint style="info" %}
After successful registration, save both the user entity and the credential source to your database. The controller automatically validates the attestation response.
{% endhint %}

## Adding Authenticators to Existing Users

Existing users can register additional authenticators (backup keys, different devices).

### Add Authenticator Form

{% code title="templates/security/add_authenticator.html.twig" lineNumbers="true" %}
```twig
<form
    action="{{ path('app_add_authenticator') }}"
    method="post"
    {{ stimulus_controller('@web-auth/webauthn-stimulus',
        {
            creationOptionsUrl: path('webauthn.controller.creation.creation.add_device'),
            creationResultField: 'input[name="attestation"]'
        }
    ) }}
>
    <input type="hidden" id="attestation" name="attestation">

    <button
        type="submit"
        {{ stimulus_action('@web-auth/webauthn-stimulus', 'register') }}
    >
        Add New Authenticator
    </button>
</form>
```
{% endcode %}

### Configuration for Existing Users

{% code title="config/packages/webauthn.yaml" lineNumbers="true" %}
```yaml
webauthn:
    controllers:
        enabled: true
        creation:
            add_device:
                options_path: '/devices/add/options'
                result_path: '/devices/add'
                user_entity_guesser: 'Webauthn\Bundle\Security\Guesser\CurrentUserEntityGuesser'
```
{% endcode %}

{% hint style="success" %}
The `CurrentUserEntityGuesser` automatically retrieves the currently authenticated user from the Symfony security context. No custom guesser needed!
{% endhint %}

## Stimulus Controller Options

The registration controller accepts the following options:

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `creationOptionsUrl` | string | Yes | URL to fetch credential creation options |
| `creationResultField` | string | Yes | CSS selector for the hidden attestation field |
| `autoRegister` | boolean | No | Automatically trigger registration on page load (default: false) |

### Auto-Register Example

Automatically start the registration process when the page loads:

{% code lineNumbers="true" %}
```twig
{{ stimulus_controller('@web-auth/webauthn-stimulus',
    {
        autoRegister: true,
        creationOptionsUrl: path('webauthn.controller.creation.creation.new_user'),
        creationResultField: 'input[name="attestation"]'
    }
) }}
```
{% endcode %}

{% hint style="warning" %}
Use `autoRegister` carefully as it immediately prompts the user for biometric authentication or security key interaction.
{% endhint %}

## Dedicated Registration Controller

{% hint style="info" %}
**New in v5.3.0:** A dedicated registration controller is available alongside the combined controller.
{% endhint %}

The dedicated `registration-controller` provides a focused controller for credential registration with enhanced features like conditional create support.

### Basic Usage

{% code title="templates/registration/register.html.twig" lineNumbers="true" %}
```twig
<form
    action="{{ path('app_register') }}"
    method="post"
    {{ stimulus_controller('@web-auth/webauthn-stimulus/registration', {
        optionsUrl: path('webauthn.controller.creation.creation.new_user'),
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
        Register with Passkey
    </button>
</form>
```
{% endcode %}

### Registration Controller Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `optionsUrl` | string | `/registration/options` | URL to fetch creation options |
| `resultUrl` | string | `/registration/verify` | URL to submit the attestation response |
| `submitViaForm` | boolean | `false` | Submit via form instead of fetch |
| `successRedirectUri` | string | - | URL to redirect after successful registration |
| `autoRegister` | boolean | `false` | Auto-start registration on page load |

### Registration Controller Targets

| Target | Description |
|--------|-------------|
| `username` | Input field for the username |
| `attestation` | Select field for attestation preference |
| `residentKey` | Select field for resident key preference |
| `userVerification` | Select field for user verification preference |
| `authenticatorAttachment` | Select field for authenticator attachment preference |
| `result` | Hidden input for the attestation response |

## Error Handling

Handle registration errors in your custom event listeners:

{% code lineNumbers="true" %}
```javascript
document.addEventListener('webauthn:registration:error', (event) => {
    console.error('Registration failed:', event.detail);
    alert('Could not register authenticator: ' + event.detail.message);
});

document.addEventListener('webauthn:registration:success', (event) => {
    console.log('Registration successful!');
});
```
{% endcode %}

## See Also

* [User Authentication](user-authentication.md) - Authenticate users with registered authenticators
* [Additional Authenticators](additional-authenticators.md) - Manage multiple authenticators
* [Bundle Configuration](../symfony-bundle/configuration-references.md) - Complete configuration reference
