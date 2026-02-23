# Additional Authenticators

Users can register multiple authenticators to their account for backup purposes or to use different devices. This page explains how to manage multiple authenticators per user.

## Why Multiple Authenticators?

Allowing users to register multiple authenticators provides several benefits:

* **Backup authenticators**: If a user loses their primary device, they can still access their account
* **Multiple devices**: Use work laptop, personal phone, and security key
* **Device upgrades**: Smoothly transition when replacing devices
* **Shared accounts**: Family members can each use their own authenticator (if your security policy allows)

## Listing User Authenticators

Display all authenticators registered to the current user:

{% code title="src/Controller/SecurityController.php" lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

namespace App\Controller;

use App\Repository\WebauthnCredentialRepository;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;
use Symfony\Component\Security\Http\Attribute\IsGranted;

#[IsGranted('ROLE_USER')]
class SecurityController extends AbstractController
{
    #[Route('/security/authenticators', name: 'app_list_authenticators')]
    public function listAuthenticators(
        WebauthnCredentialRepository $credentialRepository
    ): Response {
        $user = $this->getUser();
        $userHandle = $user->getUserIdentifier(); // Or your user ID method

        $credentials = $credentialRepository->findAllForUserEntity($userHandle);

        return $this->render('security/authenticators.html.twig', [
            'credentials' => $credentials,
        ]);
    }
}
```
{% endcode %}

{% code title="templates/security/authenticators.html.twig" lineNumbers="true" %}
```twig
{% extends 'base.html.twig' %}

{% block body %}
    <h1>My Authenticators</h1>

    {% if credentials is empty %}
        <p>No authenticators registered yet.</p>
    {% else %}
        <table>
            <thead>
                <tr>
                    <th>Authenticator ID</th>
                    <th>Transports</th>
                    <th>Counter</th>
                    <th>Registered</th>
                    <th>Actions</th>
                </tr>
            </thead>
            <tbody>
            {% for credential in credentials %}
                <tr>
                    <td>{{ credential.publicKeyCredentialId|slice(0, 16) }}...</td>
                    <td>{{ credential.transports|join(', ') }}</td>
                    <td>{{ credential.counter }}</td>
                    <td>{{ credential.createdAt|date('Y-m-d H:i') }}</td>
                    <td>
                        <a href="{{ path('app_remove_authenticator', {id: credential.id}) }}">
                            Remove
                        </a>
                    </td>
                </tr>
            {% endfor %}
            </tbody>
        </table>
    {% endif %}

    <p>
        <a href="{{ path('app_add_authenticator') }}" class="button">
            Add New Authenticator
        </a>
    </p>
{% endblock %}
```
{% endcode %}

## Adding Additional Authenticators

Allow authenticated users to register additional authenticators:

{% code title="templates/security/add_authenticator.html.twig" lineNumbers="true" %}
```twig
{% extends 'base.html.twig' %}

{% block body %}
    <h1>Add New Authenticator</h1>

    <p>
        Register a new security key, passkey, or biometric authenticator
        to use as a backup or on another device.
    </p>

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
            Register New Authenticator
        </button>

        <a href="{{ path('app_list_authenticators') }}">Cancel</a>
    </form>
{% endblock %}
```
{% endcode %}

{% hint style="info" %}
The controller configuration for `add_device` should use `CurrentUserEntityGuesser` to automatically get the authenticated user. See the [User Registration](user-registration.md#configuration-for-existing-users) page for configuration details.
{% endhint %}

## Removing Authenticators

Allow users to remove authenticators they no longer use:

{% code title="src/Controller/SecurityController.php" lineNumbers="true" %}
```php
#[Route('/security/authenticators/{id}/remove', name: 'app_remove_authenticator')]
#[IsGranted('ROLE_USER')]
public function removeAuthenticator(
    string $id,
    WebauthnCredentialRepository $credentialRepository
): Response {
    $user = $this->getUser();
    $userHandle = $user->getUserIdentifier();

    // Find the credential
    $credential = $credentialRepository->find($id);

    if (!$credential || $credential->userHandle !== $userHandle) {
        throw $this->createNotFoundException('Authenticator not found');
    }

    // Check if this is the last authenticator
    $userCredentials = $credentialRepository->findAllForUserEntity($userHandle);
    if (count($userCredentials) === 1) {
        $this->addFlash('error', 'Cannot remove your last authenticator');
        return $this->redirectToRoute('app_list_authenticators');
    }

    // Remove the authenticator
    $credentialRepository->remove($credential);

    $this->addFlash('success', 'Authenticator removed successfully');
    return $this->redirectToRoute('app_list_authenticators');
}
```
{% endcode %}

{% hint style="warning" %}
**Important Security Considerations:**

* Always verify the authenticator belongs to the current user before removal
* Prevent users from removing their last authenticator (would lock them out)
* Consider requiring re-authentication before removal for sensitive accounts
* Log authenticator additions and removals for security auditing
{% endhint %}

## Naming Authenticators

Allow users to give friendly names to their authenticators for easier management:

{% code title="src/Entity/WebauthnCredential.php" lineNumbers="true" %}
```php
use Doctrine\ORM\Mapping as ORM;
use Webauthn\CredentialRecord;

#[ORM\Entity]
class WebauthnCredential extends CredentialRecord
{
    #[ORM\Id]
    #[ORM\Column(type: 'string')]
    private string $id;

    #[ORM\Column(type: 'string', nullable: true)]
    private ?string $name = null;

    #[ORM\Column(type: 'datetime_immutable')]
    private \DateTimeImmutable $createdAt;

    #[ORM\Column(type: 'datetime_immutable', nullable: true)]
    private ?\DateTimeImmutable $lastUsedAt = null;

    public function setName(?string $name): void
    {
        $this->name = $name;
    }

    public function getName(): ?string
    {
        return $this->name;
    }

    public function updateLastUsedAt(): void
    {
        $this->lastUsedAt = new \DateTimeImmutable();
    }

    // ... other methods
}
```
{% endcode %}

Then allow users to set names during or after registration:

{% code title="templates/security/add_authenticator.html.twig" lineNumbers="true" %}
```twig
<form ...>
    <label for="authenticator_name">Authenticator Name (optional)</label>
    <input
        type="text"
        id="authenticator_name"
        name="authenticator_name"
        placeholder="e.g., My iPhone, Work Laptop, YubiKey"
    >

    <input type="hidden" id="attestation" name="attestation">

    <button type="submit">Register New Authenticator</button>
</form>
```
{% endcode %}

## Best Practices

### Encourage Backup Authenticators

Prompt users to register a backup authenticator after their first registration:

{% code lineNumbers="true" %}
```twig
{% if credential_count == 1 %}
    <div class="alert alert-info">
        <strong>Add a backup authenticator!</strong>
        Register a second authenticator to ensure you can always access your account,
        even if you lose your primary device.
        <a href="{{ path('app_add_authenticator') }}">Add backup now</a>
    </div>
{% endif %}
```
{% endcode %}

### Authenticator Metadata

Display useful information about each authenticator:

* **Transport types**: USB, NFC, Bluetooth, Internal
* **Last used date**: Help users identify unused authenticators
* **Registration date**: Track when each authenticator was added
* **AAGUID**: Identify the authenticator model (if available)

### Security Recommendations

* **Minimum authenticators**: Consider requiring at least 2 authenticators for privileged accounts
* **Maximum authenticators**: Limit to prevent abuse (e.g., 10 per user)
* **Inactive authenticators**: Automatically remove authenticators not used for a long period
* **Notification**: Email users when authenticators are added or removed

## See Also

* [User Registration](user-registration.md) - Register the first authenticator
* [User Authentication](user-authentication.md) - Authenticate with any registered authenticator
* [Advanced Behaviors](../symfony-bundle/advanced-behaviors/README.md) - Advanced configuration options
