# Signal API

{% hint style="info" %}
**Library since v5.3.0**. Symfony bundle helpers added in v5.4.0.
{% endhint %}

The [WebAuthn Signal API (W3C §5.1.10)](https://www.w3.org/TR/webauthn-3/#sctn-signal-methods) lets relying parties notify the browser's passkey manager about credential state changes. The flow is purely outbound from the page: the relying party composes the payload server-side, the page calls `navigator.credentials.signalXxx()`, and the user agent forwards the action to attached authenticators. **Nothing comes back**: the call returns `Promise<undefined>`, fire-and-forget.

## What Is the Signal API?

When users manage their passkeys on the server side (removing credentials, updating profile information), the client platform may still display outdated information. The Signal API provides a standardized way to inform the client about:

* Which credentials are still valid for a user
* Updated user details (name, display name)
* Credentials that are no longer recognized by the server

{% hint style="danger" %}
**Privacy gates per W3C §14.6.3.** Two of the three signals leak PII to whoever receives them, so they must only be exposed to authenticated callers:

| Signal | Authentication required? |
| --- | --- |
| `signalUnknownCredential` | **No**. The credential id is one the caller already presented (e.g. failed login attempt). No leak. |
| `signalAllAcceptedCredentials` | **Yes**. The full credential id list correlates a user across relying parties. |
| `signalCurrentUserDetails` | **Yes**. User handle plus name/displayName are PII. |
{% endhint %}

{% hint style="warning" %}
**Caveat on `AllAcceptedCredentials` (W3C §5.1.10.3).** Credentials missing from `allAcceptedCredentialIds` may be **removed or hidden, potentially irreversibly** by the authenticator. Always derive the list exhaustively from your truth source, never from a partial view.
{% endhint %}

## Signal Types

### AllAcceptedCredentials

Informs the client about all credentials that the server currently recognizes for a given user. This allows the client to remove any credentials that are no longer valid.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\PublicKeyCredentialDescriptor;
use Webauthn\PublicKeyCredentialRpEntity;
use Webauthn\PublicKeyCredentialUserEntity;
use Webauthn\Signal\AllAcceptedCredentials;

$rpEntity = PublicKeyCredentialRpEntity::create(id: 'example.com');

$userEntity = PublicKeyCredentialUserEntity::create(
    'john.doe',
    $userHandle,
    'John Doe'
);

// List of credential descriptors still valid for this user
$acceptedCredentials = [
    PublicKeyCredentialDescriptor::create('public-key', $credentialId1),
    PublicKeyCredentialDescriptor::create('public-key', $credentialId2),
];

$signal = new AllAcceptedCredentials($rpEntity, $userEntity, $acceptedCredentials);
```
{% endcode %}

### CurrentUserDetails

Informs the client about updated user details. Use this when a user changes their username or display name to keep the client's passkey list accurate.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\PublicKeyCredentialRpEntity;
use Webauthn\PublicKeyCredentialUserEntity;
use Webauthn\Signal\CurrentUserDetails;

$rpEntity = PublicKeyCredentialRpEntity::create(id: 'example.com');

$userEntity = PublicKeyCredentialUserEntity::create(
    'new.username',        // Updated username
    $userHandle,
    'New Display Name'     // Updated display name
);

$signal = new CurrentUserDetails($rpEntity, $userEntity);
```
{% endcode %}

### UnknownCredential

Informs the client that a specific credential is not recognized by the server. This can occur when a credential has been deleted server-side or was never registered.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\PublicKeyCredentialDescriptor;
use Webauthn\PublicKeyCredentialRpEntity;
use Webauthn\Signal\UnknownCredential;

$rpEntity = PublicKeyCredentialRpEntity::create(id: 'example.com');

$unknownCredential = PublicKeyCredentialDescriptor::create('public-key', $credentialId);

$signal = new UnknownCredential($rpEntity, $unknownCredential);
```
{% endcode %}

## Serialization

Signals can be serialized to JSON using the Symfony Serializer. The framework provides dedicated denormalizers for each signal type.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Symfony\Component\Serializer\Normalizer\AbstractObjectNormalizer;

// Serialize the signal to JSON
$json = $serializer->serialize(
    $signal,
    'json',
    [AbstractObjectNormalizer::SKIP_NULL_VALUES => true]
);

// The JSON output follows the W3C Signal API format
// For AllAcceptedCredentials:
// {
//     "rpId": "example.com",
//     "userId": "...",
//     "allAcceptedCredentialIds": ["...", "..."]
// }
```
{% endcode %}

## Use Cases

### After Credential Deletion

When a user removes a passkey from your application, send an `AllAcceptedCredentials` signal with the remaining credentials so the client can update its list.

### After Profile Update

When a user changes their username or display name, send a `CurrentUserDetails` signal so the client displays the correct information in its passkey picker.

### During Authentication

If an authentication attempt references a credential that doesn't exist in your database, send an `UnknownCredential` signal to help the client clean up orphaned passkeys.

{% hint style="info" %}
The W3C spec recommends running `signalAllAcceptedCredentials` and `signalCurrentUserDetails` **on every sign-in**: authenticators may have been detached when the underlying event (delete, profile update) fired, so the next login is the natural moment to push the up-to-date state.
{% endhint %}

## Symfony Bundle Helpers

{% hint style="info" %}
**New in v5.4.0**
{% endhint %}

The Symfony bundle ships two helpers that build the right payload from your repository and wrap them in a JSON envelope the client picks up. Wire them from your own `SuccessHandler` and `FailureHandler` (no bundle configuration), and you keep full control over *when* signals are emitted.

### Building signals with `WebauthnSignalFactory`

`Webauthn\Bundle\Service\WebauthnSignalFactory` is autowired and exposes three builders:

| Method | Purpose |
| --- | --- |
| `forUnknownCredential(string $rpId, PublicKeyCredentialDescriptor $credential)` | Build an `UnknownCredential` signal from a credential descriptor (e.g. the one a user just failed to authenticate with). |
| `forAllAccepted(string $rpId, PublicKeyCredentialUserEntity $user)` | Build an `AllAcceptedCredentials` signal. The descriptor list is derived **exhaustively** from `CredentialRecordRepositoryInterface::findAllForUserEntity()` to defuse the *"potentially irreversible"* deletion warning of W3C §5.1.10.3. |
| `forCurrentUser(string $rpId, PublicKeyCredentialUserEntity $user)` | Build a `CurrentUserDetails` signal carrying the user's current `name` and `displayName`. |

### Wrapping the response with `WebauthnSignalResponse::withSignals()`

`Webauthn\Bundle\Service\WebauthnSignalResponse` produces a `JsonResponse` that combines your application payload with a top-level `signals: [{type, options}]` envelope:

{% code lineNumbers="true" %}
```json
{
    "your": "payload",
    "signals": [
        {
            "type": "allAcceptedCredentials",
            "options": {
                "rpId": "example.com",
                "userId": "dXNlci1oYW5kbGUtYnl0ZXM",
                "allAcceptedCredentialIds": ["Y3JlZC1B", "Y3JlZC1C"]
            }
        },
        {
            "type": "currentUserDetails",
            "options": {
                "rpId": "example.com",
                "userId": "dXNlci1oYW5kbGUtYnl0ZXM",
                "name": "alice@example.com",
                "displayName": "Alice Liddell"
            }
        }
    ]
}
```
{% endcode %}

The Stimulus base controller picks the envelope up automatically (see [Stimulus Integration](#stimulus-integration) below).

### Emit on every sign-in: custom `SuccessHandler`

The W3C spec recommends emitting `allAcceptedCredentials` and `currentUserDetails` on every sign-in so authenticators that were detached when the change happened catch up later. A focused `SuccessHandler` does exactly that:

{% code title="src/Security/WebauthnSignalingSuccessHandler.php" lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

namespace App\Security;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Webauthn\Bundle\Security\Handler\SuccessHandler;
use Webauthn\Bundle\Service\WebauthnSignalFactory;
use Webauthn\Bundle\Service\WebauthnSignalResponse;
use Webauthn\PublicKeyCredential;
use Webauthn\PublicKeyCredentialOptions;
use Webauthn\PublicKeyCredentialUserEntity;

final readonly class WebauthnSignalingSuccessHandler implements SuccessHandler
{
    private const RP_ID = 'example.com';

    public function __construct(
        private WebauthnSignalFactory $signals,
        private WebauthnSignalResponse $response,
    ) {
    }

    public function onSuccess(
        Request $request,
        ?PublicKeyCredential $publicKeyCredential = null,
        ?PublicKeyCredentialOptions $publicKeyCredentialOptions = null,
        ?PublicKeyCredentialUserEntity $userEntity = null,
    ): Response {
        if ($userEntity === null) {
            return $this->response->withSignals(['status' => 'ok']);
        }

        return $this->response->withSignals(
            ['status' => 'ok'],
            $this->signals->forAllAccepted(self::RP_ID, $userEntity),
            $this->signals->forCurrentUser(self::RP_ID, $userEntity),
        );
    }
}
```
{% endcode %}

### Drop stale credentials: custom `FailureHandler`

When an assertion fails because the relying party no longer recognises the credential id presented, emit `signalUnknownCredential` so the user agent can drop the stale passkey from its picker.

The bundle's response controllers pass three optional arguments to `FailureHandler::onFailure` (the deserialized `PublicKeyCredential`, the stored options, and the resolved user entity) so the handler can build a signal without re-parsing the request. Add them to your signature:

{% code title="src/Security/WebauthnSignalingFailureHandler.php" lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

namespace App\Security;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Throwable;
use Webauthn\Bundle\Security\Handler\FailureHandler;
use Webauthn\Bundle\Service\WebauthnSignalFactory;
use Webauthn\Bundle\Service\WebauthnSignalResponse;
use Webauthn\Exception\AuthenticatorResponseVerificationException;
use Webauthn\PublicKeyCredential;
use Webauthn\PublicKeyCredentialOptions;
use Webauthn\PublicKeyCredentialUserEntity;

final readonly class WebauthnSignalingFailureHandler implements FailureHandler
{
    private const RP_ID = 'example.com';

    public function __construct(
        private WebauthnSignalFactory $signals,
        private WebauthnSignalResponse $response,
    ) {
    }

    public function onFailure(
        Request $request,
        ?Throwable $exception = null,
        ?PublicKeyCredential $publicKeyCredential = null,
        ?PublicKeyCredentialOptions $publicKeyCredentialOptions = null,
        ?PublicKeyCredentialUserEntity $userEntity = null,
    ): Response {
        $signals = [];

        if (
            $publicKeyCredential !== null
            && $exception instanceof AuthenticatorResponseVerificationException
        ) {
            $signals[] = $this->signals->forUnknownCredential(
                self::RP_ID,
                $publicKeyCredential->getPublicKeyCredentialDescriptor(),
            );
        }

        return $this->response->withSignals(
            ['status' => 'error', 'errorMessage' => $exception?->getMessage() ?? 'Unexpected error'],
            ...$signals,
        );
    }
}
```
{% endcode %}

{% hint style="info" %}
**Why the extra arguments?** In v5.x the `FailureHandler::onFailure` interface only declares `Request` and `?Throwable`. The three additional optional arguments (`?PublicKeyCredential`, `?PublicKeyCredentialOptions`, `?PublicKeyCredentialUserEntity`) are passed by the bundle's response controllers but live as a PHPDoc-only signature on the interface for backwards compatibility. Implementations that have not updated their signature ignore the extras silently. They will become required parameters in 6.0.
{% endhint %}

### Wiring

Both helpers are autowired. Reference your handlers from the firewall configuration:

{% code title="config/packages/security.yaml" %}
```yaml
security:
    firewalls:
        main:
            webauthn:
                authentication:
                    success_handler: App\Security\WebauthnSignalingSuccessHandler
                    failure_handler: App\Security\WebauthnSignalingFailureHandler
```
{% endcode %}

## Stimulus Integration

The `@web-auth/webauthn-stimulus` controllers will pick up the `signals: [...]` envelope automatically and dispatch each entry to the matching `PublicKeyCredential.signalXxx()` JS API (feature-detected, silently no-op on browsers that do not expose the method).

{% hint style="info" %}
**Browser support.** The Signal API requires Chrome/Edge ≥ 128, Safari ≥ 18.4 (or 18.5 for the `allAcceptedCredentials` and `currentUserDetails` variants). Firefox does not implement it yet. The Stimulus dispatch is intentionally fire-and-forget so the rest of your application flow proceeds unchanged on unsupported user agents.
{% endhint %}

## See Also

* [WebAuthn L3 §5.1.10, Signal Methods](https://www.w3.org/TR/webauthn-3/#sctn-signal-methods): the W3C specification.
* [WebAuthn L3 §14.6.3, Privacy leak via credential IDs](https://www.w3.org/TR/webauthn-3/#sctn-credential-id-privacy-leak): the rationale behind the privacy gates above.
* [Credential Record](../../prerequisites/credential-record.md): credential storage.
