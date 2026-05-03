# PRF Extension

{% hint style="info" %}
**New in v5.4.0**
{% endhint %}

The [WebAuthn Pseudo-Random Function (PRF) extension](https://www.w3.org/TR/webauthn-3/#prf-extension) lets the relying party derive an authenticator-bound, salt-bound secret on the client. Because the secret is a deterministic function of `(credential, salt)`, it is the building block for **client-side encryption**: the authenticator returns the same bytes for the same salt as long as the credential lives, the relying party never sees them, and yet the user can decrypt their own data after a fresh assertion.

The framework ships a fluent builder for the `prf` extension input plus first-class handling on the Stimulus side. The output is left to the application — what you do with `clientExtensionResults.prf.results.first` (typically: feed it through HKDF and use the output as an AES-GCM key) is yours to design.

{% hint style="danger" %}
**A passkey deletion permanently destroys all PRF-protected data.** Every passkey owns its own PRF input-key inside the authenticator. The relying party never sees that key, the user agent never sees that key, and there is no API to export it. Re-creating a passkey produces a brand-new input-key, so anything sealed under the previous passkey is irrecoverable.

Production use of PRF therefore needs an explicit recovery story — a second passkey enrolled with shared salts, a server-side wrap escrowed behind a strong second factor, or a "you will lose this data if you lose this passkey" UX contract the user accepts. See Matt Miller's [May 2025 update](https://blog.millerti.me/2023/01/22/encrypting-data-in-the-browser-using-webauthn/) for the full rationale.
{% endhint %}

## Building a PRF Extension

`Webauthn\AuthenticationExtensions\PseudoRandomFunctionInputExtensionBuilder` is the entry point. It produces a `PseudoRandomFunctionInputExtension` ready to be attached to your creation or request options.

### Global Salts (`eval`)

Use `withInputs()` for the WebAuthn `eval` block — the canonical form during registration when the credential id is not yet known, and a usable fallback during authentication for credentials not covered by `evalByCredential`.

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\AuthenticationExtensions\AuthenticationExtensions;
use Webauthn\AuthenticationExtensions\PseudoRandomFunctionInputExtensionBuilder;
use Webauthn\PublicKeyCredentialCreationOptions;

$prf = PseudoRandomFunctionInputExtensionBuilder::create()
    ->withInputs(random_bytes(32))                  // raw salt, base64url-encoded for transport
    ->withInputs(random_bytes(32), random_bytes(32))  // optional second salt → results.second
    ->build();

$options = PublicKeyCredentialCreationOptions::create(
    rp: $rpEntity,
    user: $userEntity,
    challenge: random_bytes(32),
    extensions: AuthenticationExtensions::create([$prf]),
);
```
{% endcode %}

The salts are passed in **raw bytes**. The builder takes care of base64url-encoding them so the JSON sent to the browser matches the W3C wire format.

### Per-credential Salts (`evalByCredential`)

During authentication, prefer `withCredentialInputs()` so each credential is queried with its own salt — for instance a salt rotated alongside a re-encrypted blob:

{% code lineNumbers="true" %}
```php
$prf = PseudoRandomFunctionInputExtensionBuilder::create();
foreach ($userCredentials as $credentialRecord) {
    $prf = $prf->withCredentialInputs(
        credentialId: $credentialRecord->publicKeyCredentialId,  // raw credential id bytes
        first: $saltStore->saltFor($credentialRecord),
    );
}

$options = PublicKeyCredentialRequestOptions::create(
    challenge: random_bytes(32),
    rpId: 'example.com',
    allowCredentials: $allowList,
    extensions: AuthenticationExtensions::create([$prf->build()]),
);
```
{% endcode %}

{% hint style="info" %}
The W3C spec keys `evalByCredential` by the **base64url string** of the credential id. `withCredentialInputs()` accepts the raw credential id bytes (the same bytes you store on the `CredentialRecord`) and encodes them for you. If you only hold the base64url form, pass it as-is — the builder will not double-encode.
{% endhint %}

### Validation

Calling `build()` without ever calling `withInputs()` or `withCredentialInputs()` throws `AuthenticationExtensionException`:

```text
Cannot build a PRF extension without any input. Call withInputs() or withCredentialInputs() first.
```

This catches a silent failure mode where the browser would simply ignore an empty `prf` block, leaving the relying party convinced PRF was active when it never was.

### Detecting the `hmac-secret-mc` requirement

CTAP 2.2 introduced `hmac-secret-mc`, the PRF authenticator extension variant that supports PRF evaluation at credential creation time and multi-credential `evalByCredential` queries. The wire format is unchanged — the user agent picks `hmac-secret` vs `hmac-secret-mc` based on the inputs you ship.

`PseudoRandomFunctionInputExtensionBuilder::requiresHmacSecretMc()` returns `true` when the builder carries `evalByCredential` entries for more than one credential — the case the builder can detect on its own:

{% code lineNumbers="true" %}
```php
$builder = PseudoRandomFunctionInputExtensionBuilder::create()
    ->withCredentialInputs($credIdA, $saltA)
    ->withCredentialInputs($credIdB, $saltB);

if ($builder->requiresHmacSecretMc()) {
    // Authenticator must speak hmac-secret-mc; surface a fallback flow if it does not.
}
```
{% endcode %}

{% hint style="warning" %}
The create-time eval case (`eval` set during a registration ceremony) also requires `hmac-secret-mc`, but the builder cannot tell which ceremony its output will attach to — it is the caller's responsibility to flag that case at the call site.
{% endhint %}

## Reading the PRF Output

The browser surfaces the PRF result in the assertion's `clientExtensionResults.prf.results.first` (and optionally `.second`) as `ArrayBuffer`. Verifying the assertion through `AuthenticatorAssertionResponseValidator::check(...)` does not consume the output — you read it from the deserialized `PublicKeyCredential->response->clientExtensionResults` after validation succeeds and pipe it into your application-level KDF.

## Stimulus Integration

The new `authentication-controller` and `registration-controller` (and the framework's `BaseController` they extend) handle PRF transparently:

- **On the way out**, `extensions.prf.eval.{first,second}` and `extensions.prf.evalByCredential[*]` are decoded from base64url to `ArrayBuffer` before the `navigator.credentials.create()` / `get()` call.
- **On the way back**, `clientExtensionResults.prf.results.{first,second}` are re-encoded to base64url on the credential dispatched via the controller event so your verifier receives a clean JSON payload.

No additional configuration is required — return options with a `prf` extension and the controllers do the right thing.

{% hint style="warning" %}
The legacy combined controller (`@web-auth/webauthn-stimulus`) had two bugs in the legacy code path that were fixed in v5.4.0: it passed the wrong sub-object to the PRF input importer, and it read `prf.result` (singular) instead of `prf.results`. Make sure you are on v5.4.0 if you depend on PRF on the legacy controller; the dedicated `authentication`/`registration` controllers were always correct.
{% endhint %}

## Symfony Bundle

PRF support currently lives only at the library level. The Symfony bundle does not yet expose a `prf:` profile flag — to enable PRF on a bundle-driven flow you have to attach the extension yourself, for instance from a custom `CreationOptionsHandler` / `RequestOptionsHandler`:

{% code lineNumbers="true" %}
```php
public function onCreationOptions(
    PublicKeyCredentialCreationOptions $options,
    PublicKeyCredentialUserEntity $userEntity,
    ?Request $request = null,
): Response {
    $options->extensions[] = PseudoRandomFunctionInputExtensionBuilder::create()
        ->withInputs($this->saltProvider->saltFor($userEntity))
        ->build();

    return new JsonResponse($options);
}
```
{% endcode %}

A `prf:` profile block (with a pluggable `PrfSaltProviderInterface` for per-user/per-credential salt rotation) is on the roadmap for a future minor.

## Runnable Demo

The framework repository ships a complete client-side encryption demo under [`docs/examples/prf-demo/`](https://github.com/web-auth/webauthn-framework/tree/5.4.x/docs/examples/prf-demo) showing the typical PRF use case end-to-end:

* `register.html` — registers a credential with `prf.eval` primed and persists the salts next to the credential server-side.
* `vault.html` — re-issues the per-credential salts during authentication, derives an AES-GCM key from `prf.results.first` and an HMAC-SHA256 key from `prf.results.second` (HKDF, disjoint info strings), and stores AES-GCM-encrypted items + per-item HMAC server-side. The server only ever sees ciphertexts.
* `offline.html` — same flow without a server, demonstrating airplane-mode decryption via a service worker cache and `localStorage` ciphertexts.

Run it with `./docs/examples/prf-demo/same-origin/run.sh`.

## See Also

* [W3C WebAuthn PRF Extension](https://www.w3.org/TR/webauthn-3/#prf-extension)
* [Extensions](extensions.md) — generic mechanism behind `PseudoRandomFunctionInputExtension`
