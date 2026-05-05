# Verification Helpers (Custom Controllers)

{% hint style="info" %}
**New in v5.4.0**
{% endhint %}

Companion of the [Options Helpers](options-helpers.md) for the **response side**: instead of routing your `/attestation/response` and `/assertion/response` endpoints through the bundle's `AttestationResponseController` / `AssertionResponseController` (driven by `creation_profiles` / `request_profiles` config), you write your own controllers and call a single helper to deserialise the body, validate the ceremony and return a typed result. Your controller stays in charge of what to do with that result (login, redirect, JSON envelope, Signal API payload, etc.).

## The entry point

`Webauthn\Bundle\Service\WebauthnResponseVerifier` is autowired and public. It exposes two factory methods, one per ceremony, returning a fluent verifier:

```php
$this->verifier->forAttestation(string $rpId): WebauthnAttestationVerifier
$this->verifier->forAssertion(string $rpId): WebauthnAssertionVerifier
```

The terminal call `->verify($request)` returns a `WebauthnVerificationResult` value object exposing:

* `credentialRecord` (`CredentialRecord`) the validated credential record (already persisted on attestation, updated in place on assertion)
* `publicKeyCredential` (`PublicKeyCredential`) the deserialised request body, useful for the Signal API or contextual logging
* `userEntity` (`?PublicKeyCredentialUserEntity`) the user entity loaded from `OptionsStorage`, when one was associated with the ceremony

### Defaults

* On attestation, the validated credential record is persisted automatically through your `CredentialRecordRepositoryInterface` (which must also implement `CanSaveCredentialRecord`). Disable with `withSaveCredential(false)` if your controller wants to own persistence.
* On attestation, if the stored creation options carry the W3C `mediation: conditional` flag, the verifier automatically switches to the *conditional creation* ceremony manager (which relaxes User Verification per the spec). Nothing extra to do on the controller side.
* On assertion, the credential matching the response's `rawId` is fetched via `CredentialRecordRepositoryInterface::findOneByCredentialId()`. Counter, backup state and `uvInitialized` are updated in place; persistence of these updates is left to the repository (Doctrine flushes through the unit of work, exactly like the legacy controller).

## Minimal controllers

### Registration response

{% code title="src/Controller/Webauthn/RegisterResponseController.php" lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

namespace App\Controller\Webauthn;

use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\Attribute\Route;
use Webauthn\Bundle\Security\Authentication\Exception\WebauthnAuthenticationFailureException;
use Webauthn\Bundle\Service\WebauthnResponseVerifier;

final class RegisterResponseController
{
    public function __construct(
        private readonly WebauthnResponseVerifier $verifier,
    ) {
    }

    #[Route('/webauthn/register', methods: ['POST'])]
    public function __invoke(Request $request): JsonResponse
    {
        try {
            $result = $this->verifier
                ->forAttestation('example.com')
                ->verify($request);
        } catch (WebauthnAuthenticationFailureException $exception) {
            return new JsonResponse(['error' => $exception->getMessage()], 401);
        }

        return new JsonResponse([
            'status' => 'registered',
            'credentialId' => base64_encode($result->credentialRecord->publicKeyCredentialId),
        ]);
    }
}
```
{% endcode %}

### Authentication response

{% code title="src/Controller/Webauthn/LoginResponseController.php" lineNumbers="true" %}
```php
#[Route('/webauthn/login', methods: ['POST'])]
public function __invoke(Request $request): Response
{
    try {
        $result = $this->verifier
            ->forAssertion('example.com')
            ->verify($request);
    } catch (WebauthnAuthenticationFailureException $exception) {
        return new JsonResponse(['error' => $exception->getMessage()], 401);
    }

    // Log the user in. Up to you: WebauthnToken, your own UserChecker, etc.
    $this->loginUser($result->credentialRecord, $result->userEntity);

    return new JsonResponse(['status' => 'authenticated']);
}
```
{% endcode %}

`allowCredentials`, the user handle, the host and the challenge are all checked under the hood through the underlying validators and ceremony steps. Nothing else to wire on the controller side.

## Configuring the ceremony

Most of the ceremony configuration has already happened on the *options* side; the verifier exposes a small surface to override the bits that matter at validation time.

### Common (both attestation and assertion)

| Setter | Default |
| --- | --- |
| `withAllowedOrigins(string ...)` | the global `webauthn.allowed_origins` configuration |
| `withAllowSubdomains(bool = true)` | the global `webauthn.allow_subdomains` configuration |

Per-verifier override of the accepted origins. When set, the verifier asks the autowired `CeremonyStepManagerFactory` to produce a fresh `CeremonyStepManager` and a fresh validator on top, so the singleton factory's state stays untouched. Use it when one route accepts a different list of origins than the rest of the application (multi-tenant, internal-vs-public, staging vs production, etc.).

```php
$result = $this->verifier
    ->forAttestation('example.com')
    ->withAllowedOrigins('https://app.example.com', 'https://admin.example.com')
    ->withAllowSubdomains(true)
    ->verify($request);
```

### Attestation only

| Setter | Default |
| --- | --- |
| `withSaveCredential(bool = true)` | `true` (auto-persist via `CanSaveCredentialRecord`) |

If your `CredentialRecordRepositoryInterface` does not implement `CanSaveCredentialRecord`, calling `verify()` with the default behaviour throws `Webauthn\Bundle\Exception\MissingFeatureException`. Implement the interface or pass `withSaveCredential(false)` and persist yourself.

## Failure handling

`verify()` throws in two well-defined ways:

* `Symfony\Component\HttpKernel\Exception\BadRequestHttpException` for **malformed inputs**: wrong content type, empty body, deserialisation failure, unknown challenge in `OptionsStorage`, response type mismatch (assertion body posted to the attestation verifier or vice-versa), Relying Party ID mismatch. Symfony's HTTP layer turns these into HTTP 400 automatically; you don't have to catch them.
* `Webauthn\Bundle\Security\Authentication\Exception\WebauthnAuthenticationFailureException` for **ceremony validation failures**: signature invalid, user handle mismatch, attestation rejected, etc. The exception carries the deserialised `PublicKeyCredential`, the underlying `AuthenticatorResponse`, the `PublicKeyCredentialOptions` it was checked against and the user entity (when known), so your controller can build a contextual response, including the [Signal API](../pure-php/advanced-behaviours/signal-api.md) `signalUnknownCredential` payload to drop a stale passkey from the platform UI.

```php
try {
    $result = $this->verifier->forAssertion('example.com')->verify($request);
} catch (WebauthnAuthenticationFailureException $exception) {
    $signal = $this->signalFactory->forUnknownCredentialFromException('example.com', $exception);

    return $this->signalResponse->withSignals(
        ['error' => 'authentication failed'],
        ...array_filter([$signal])
    );
}
```

## Storage

The helper reads ceremony options from the bundle's `OptionsStorage` automatically (the same one [Options Helpers](options-helpers.md) write to). Use the bundle's helpers on both ends and you don't have to manage challenge storage at all.

## Why a single entry point?

`WebauthnResponseVerifier` is the only service you inject in your controllers. The two verifiers it returns are typed (`WebauthnAttestationVerifier` and `WebauthnAssertionVerifier`), so the IDE autocomplete only proposes the methods that make sense for the ceremony at hand.

```php
$this->verifier->forAttestation(...)->withSaveCredential(false);    // OK: attestation has it
$this->verifier->forAssertion(...)->withSaveCredential(false);      // ERROR: not on assertion
```

## Migration from `AttestationResponseController` / `AssertionResponseController`

If you are still routing `/attestation/response` and `/assertion/response` through the bundle's controllers via `webauthn.controllers.creation.<name>` / `webauthn.controllers.request.<name>`, the migration path is straightforward:

1. Write a controller of your own that calls `WebauthnResponseVerifier`.
2. Re-bind the existing route(s) to your controller via the `#[Route]` attribute.
3. Drop the matching `webauthn.controllers.*` config block once the migration is complete.

Your previous `SuccessHandler` / `FailureHandler` logic moves into the controller body: do whatever you need with the `WebauthnVerificationResult` (or with the caught `WebauthnAuthenticationFailureException`) and return the response yourself.

## See Also

* [Options Helpers](options-helpers.md): the matching helper for the request/options phase
* [Signal API](../pure-php/advanced-behaviours/signal-api.md): how to use the `WebauthnAuthenticationFailureException` context to push `signalUnknownCredential` payloads back to the client
* [Conditional Create](../pure-php/advanced-behaviours/conditional-create.md): the ceremony the verifier auto-detects when stored options carry `mediation: conditional`
