# Authenticator Extensions (CTAP 2.1 / FIDO Alliance)

{% hint style="info" %}
**New in v5.4.0**
{% endhint %}

WebAuthn delegates a number of authenticator-side capabilities to extensions defined by the FIDO Alliance CTAP 2.1 specification rather than by the W3C document. The framework ships first-class builders and typed output value objects for the four most common ones: `credProtect`, `credBlob` / `getCredBlob`, `minPinLength`, and the WebAuthn Level 3 update to `credProps`.

All of these live under the `Webauthn\AuthenticationExtensions` namespace and plug into the regular extensions bag — see [Extensions](extensions.md) for the generic mechanism.

## credProtect (CTAP 2.1 §12.1)

Lets the relying party request a credential-protection policy at registration time. The W3C binding exposes it as the `credentialProtectionPolicy` string (with the optional `enforceCredentialProtectionPolicy` companion); the authenticator returns the policy actually applied as a CBOR uint inside `authData.extensions.credProtect`.

### Requesting a policy

`CredentialProtectionInputExtension` exposes named factories for the three policies plus the optional enforce flag:

{% code lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

use Webauthn\AuthenticationExtensions\AuthenticationExtensions;
use Webauthn\AuthenticationExtensions\CredentialProtectionInputExtension;

$extensions = AuthenticationExtensions::create([
    CredentialProtectionInputExtension::userVerificationRequired(),
    CredentialProtectionInputExtension::enforce(),  // optional companion
]);
```
{% endcode %}

| Factory | Policy |
| --- | --- |
| `userVerificationOptional()` | `1` — UV optional |
| `userVerificationOptionalWithCredentialIDList()` | `2` — UV optional unless the credential is discoverable |
| `userVerificationRequired()` | `3` — UV always required |
| `enforce()` | `enforceCredentialProtectionPolicy: true` — ask the user agent to fail rather than silently downgrade |

### Reading the applied policy

`CredentialProtectionOutput::fromExtensions(...)` parses the uint that the authenticator wrote back. Compare it with the requested policy to detect silent downgrades on authenticators that do not honour `credProtect`:

{% code lineNumbers="true" %}
```php
use Webauthn\AuthenticationExtensions\CredentialProtectionOutput;

$applied = CredentialProtectionOutput::fromExtensions($authenticatorData->extensions);
if ($applied?->policy !== CredentialProtectionInputExtension::POLICY_UV_REQUIRED) {
    // Authenticator silently downgraded; reject or warn the user.
}
```
{% endcode %}

## credBlob / getCredBlob (CTAP 2.1 §12.2)

Stores up to 32 bytes of opaque data alongside a credential at registration time, then asks for those bytes back during a later assertion.

### Storing a blob at registration

{% code lineNumbers="true" %}
```php
use Webauthn\AuthenticationExtensions\AuthenticationExtensions;
use Webauthn\AuthenticationExtensions\CredentialBlobInputExtension;

$extensions = AuthenticationExtensions::create([
    CredentialBlobInputExtension::withBlob($rawBytes),  // raw bytes ≤ 32 — base64url'd for transport
]);
```
{% endcode %}

`withBlob()` rejects payloads above 32 bytes at construction time. The success flag the authenticator returns is exposed by `CredentialBlobRegistrationOutput`.

### Retrieving the blob at assertion

{% code lineNumbers="true" %}
```php
use Webauthn\AuthenticationExtensions\AuthenticationExtensions;
use Webauthn\AuthenticationExtensions\CredentialBlobAssertionOutput;
use Webauthn\AuthenticationExtensions\GetCredentialBlobInputExtension;

$extensions = AuthenticationExtensions::create([
    GetCredentialBlobInputExtension::enable(),
]);

// after validation:
$blob = CredentialBlobAssertionOutput::fromExtensions($authenticatorData->extensions);
$bytes = $blob?->bytes;  // raw bytes, ≤ 32
```
{% endcode %}

The Stimulus base controller takes care of encoding/decoding transparently: registration `extensions.credBlob` strings are base64url-decoded to `ArrayBuffer` before `navigator.credentials.create()`, and the assertion `clientExtensionResults.credBlob` `ArrayBuffer` is re-encoded to base64url before the credential event is dispatched. JSON sent to the server round-trips cleanly through `CredentialBlobAssertionOutput`.

## minPinLength (CTAP 2.1 §12.4)

Enabled at registration time. When the relying party id is on the authenticator's enterprise allow-list, the authenticator returns the configured minimum PIN length as a CBOR uint inside `authData.extensions.minPinLength`.

{% code lineNumbers="true" %}
```php
use Webauthn\AuthenticationExtensions\AuthenticationExtensions;
use Webauthn\AuthenticationExtensions\MinPinLengthInputExtension;
use Webauthn\AuthenticationExtensions\MinPinLengthOutput;

$extensions = AuthenticationExtensions::create([
    MinPinLengthInputExtension::enable(),
]);

// after validation:
$minPinLength = MinPinLengthOutput::fromExtensions($authenticatorData->extensions);
$value = $minPinLength?->minPinLength;  // int
```
{% endcode %}

`MinPinLengthOutput::fromExtensions(...)` rejects non-integer or negative values defensively.

## credProps and `authenticatorDisplayName` (WebAuthn L3 §10.1.3)

WebAuthn Level 3 adds an `authenticatorDisplayName` field next to the existing `rk` flag in the `credProps` client extension output. The framework now exposes a typed value object plus a checker that structurally validates the output when the extension was requested.

{% code lineNumbers="true" %}
```php
use Webauthn\AuthenticationExtensions\AuthenticationExtension;
use Webauthn\AuthenticationExtensions\AuthenticationExtensions;
use Webauthn\AuthenticationExtensions\CredentialPropertiesOutput;
use Webauthn\AuthenticationExtensions\CredentialPropertiesOutputChecker;

// Request the extension
$extensions = AuthenticationExtensions::create([
    AuthenticationExtension::create('credProps', true),
]);

// After validation, read the typed view
$credProps = CredentialPropertiesOutput::fromExtensions($clientExtensionResults);
$isPasskey         = $credProps?->rk;                       // bool|null
$authenticatorName = $credProps?->authenticatorDisplayName; // string|null
```
{% endcode %}

Register `CredentialPropertiesOutputChecker` with your `ExtensionOutputCheckerHandler` (see [Extensions](extensions.md)) to enforce the `boolean rk` / `string authenticatorDisplayName` shape automatically when the extension is in use.

## See Also

* [W3C WebAuthn L3 §10.1.3](https://www.w3.org/TR/webauthn-3/#sctn-authenticator-credential-properties-extension) — `credProps` extension
* [CTAP 2.1 §12.1](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#sctn-credProtect-extension) — `credProtect`
* [CTAP 2.1 §12.2](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#sctn-credBlob-extension) — `credBlob` / `getCredBlob`
* [CTAP 2.1 §12.4](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html#sctn-minpinlength-extension) — `minPinLength`
* [Extensions](extensions.md) — generic extension mechanism
* [PRF Extension](prf-extension.md) — also a CTAP-bound extension (`hmac-secret` / `hmac-secret-mc`)
