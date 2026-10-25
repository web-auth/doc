---
description: Prove to me who you claim to be!
---

# User Verification

User verification is a WebAuthn feature that confirms the identity of the actual user, not just possession of the authenticator. It answers the question: "Is the person using this authenticator really the authorized user?"

## What is User Verification?

User verification requires the user to prove their identity through:

* 🔐 **PIN code** - Entering a secret code on the authenticator
* 👆 **Biometric recognition** - Fingerprint, facial recognition, iris scan
* 🔑 **Password entry** - Entering a password on the device
* 🔒 **Combined methods** - Touch + PIN, or other combinations

The goal is to **distinguish individual users** and ensure that the person authenticating is the legitimate account owner, not just someone who stole or found the authenticator.

## User Presence vs User Verification

It's important to understand the difference:

| Feature | User Presence (UP) | User Verification (UV) |
|---------|-------------------|----------------------|
| **What it proves** | Someone is there | **Who** is there |
| **How** | Simple button press, touch | PIN, biometric, password |
| **Flag** | `UP` flag in authenticator data | `UV` flag in authenticator data |
| **Security level** | Basic | Enhanced |
| **Requirement** | Always required | Optional (configurable) |

**User Presence** is always enforced by WebAuthn - it simply confirms someone physically interacted with the authenticator (like touching a button).

**User Verification** goes further by confirming the specific identity of that person.

## User Verification Requirements

You can configure the user verification requirement when creating credential options. There are three possible values:

### `required`

The operation **requires** user verification and will **fail** if the authenticator does not set the `UV` flag.

{% code lineNumbers="true" %}
```php
use Webauthn\AuthenticatorSelectionCriteria;
use Webauthn\PublicKeyCredentialCreationOptions;

$authenticatorSelection = AuthenticatorSelectionCriteria::create(
    userVerification: AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_REQUIRED
);

$publicKeyCredentialCreationOptions = PublicKeyCredentialCreationOptions::create(
    rp: $rpEntity,
    user: $userEntity,
    challenge: random_bytes(32),
    authenticatorSelection: $authenticatorSelection
);
```
{% endcode %}

**Use when:**
* High-security applications (banking, healthcare)
* Privileged operations (admin actions, financial transactions)
* Compliance requirements mandate biometric authentication

### `preferred`

The operation **prefers** user verification but will **not fail** if the authenticator doesn't provide it.

{% code lineNumbers="true" %}
```php
$authenticatorSelection = AuthenticatorSelectionCriteria::create(
    userVerification: AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED
);
```
{% endcode %}

This is the **recommended default** for most applications. It uses user verification when available but doesn't exclude authenticators that don't support it.

**Use when:**
* General-purpose authentication
* You want security when available without limiting device compatibility
* Balancing security and user experience

### `discouraged`

The operation **discourages** user verification to minimize disruption to the user flow.

{% code lineNumbers="true" %}
```php
$authenticatorSelection = AuthenticatorSelectionCriteria::create(
    userVerification: AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_DISCOURAGED
);
```
{% endcode %}

**Use when:**
* Low-risk operations
* Maximizing speed and convenience
* Simple presence verification is sufficient
* Testing or demonstration purposes

## Checking the UV Flag

The `userVerification` setting in the credential options is a **request** sent to the authenticator, not a guarantee. The `CheckUserVerification` ceremony step enforces it against the options that were actually emitted to the client — so if those options were built with a weaker requirement than your profile (for example, because [Client Override Policy](../symfony-bundle/advanced-behaviors/client-override-policy.md) is configured to allow a client to lower it), the ceremony will pass even though no verification took place.

{% hint style="warning" %}
**Mandatory for sensitive operations.** Do not rely solely on `userVerification: required` in the profile to gate access to privileged actions (admin, payment, secret rotation, AAL2/AAL3 compliance). Always re-check the `UV` flag on the returned authenticator data after a successful ceremony. This is your final, authoritative signal that user verification actually occurred.
{% endhint %}

After authentication or registration, inspect the flag on the response's authenticator data and decide whether to proceed:

{% code lineNumbers="true" %}
```php
// $authenticatorResponse is the AuthenticatorAssertionResponse or
// AuthenticatorAttestationResponse that was just successfully validated.
$authenticatorData = $authenticatorResponse instanceof AuthenticatorAssertionResponse
    ? $authenticatorResponse->authenticatorData
    : $authenticatorResponse->attestationObject->authData;

if (! $authenticatorData->isUserVerified()) {
    // Only user presence was confirmed (button press / touch).
    // Reject for sensitive flows even if the ceremony succeeded.
    throw new AccessDeniedHttpException('User verification is required for this action.');
}

// Safe to proceed with the privileged operation
```
{% endcode %}

You can also read the raw flags if you need fine-grained checks (for example combining `UV` with `BS`/`BE` for backup-eligibility policies):

{% code lineNumbers="true" %}
```php
$flags = $authenticatorData->getFlags();
$uvFlag = (ord($flags) & 0x04) !== 0;  // UV flag is bit 2
```
{% endcode %}

## Practical Examples

### Example 1: High-Security Banking App

{% code lineNumbers="true" %}
```php
// Require user verification for all operations
$authenticatorSelection = AuthenticatorSelectionCriteria::create(
    userVerification: AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_REQUIRED
);

// This ensures only authenticators with PIN/biometric can be used
$options = PublicKeyCredentialCreationOptions::create(
    rp: $rpEntity,
    user: $userEntity,
    challenge: random_bytes(32),
    authenticatorSelection: $authenticatorSelection
);
```
{% endcode %}

### Example 2: General Web Application

{% code lineNumbers="true" %}
```php
// Prefer user verification but don't require it
$authenticatorSelection = AuthenticatorSelectionCriteria::create(
    userVerification: AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED
);

// Works with all authenticators, uses UV when available
$options = PublicKeyCredentialCreationOptions::create(
    rp: $rpEntity,
    user: $userEntity,
    challenge: random_bytes(32),
    authenticatorSelection: $authenticatorSelection
);
```
{% endcode %}

### Example 3: Fast Re-authentication

{% code lineNumbers="true" %}
```php
// For quick re-auth on already logged-in user, discourage UV
$options = PublicKeyCredentialRequestOptions::create(
    challenge: random_bytes(32),
    userVerification: AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_DISCOURAGED
);
```
{% endcode %}

## Device Capabilities

Different authenticators have different user verification capabilities:

| Authenticator Type | User Verification Methods |
|-------------------|--------------------------|
| **Touch ID / Face ID (macOS/iOS)** | Fingerprint, facial recognition |
| **Windows Hello** | Fingerprint, facial recognition, PIN |
| **Android Platform** | Fingerprint, facial recognition, pattern, PIN |
| **YubiKey 5 Series** | PIN (must be set up) |
| **YubiKey Bio** | Fingerprint |
| **Basic security keys** | None (only user presence) |

{% hint style="info" %}
When you set `userVerification: required`, only authenticators capable of user verification will be available to the user during registration or authentication.
{% endhint %}

## Best Practices

### For Registration

1. **Use `preferred` as default** - Balances security and compatibility
2. **Use `required` for sensitive accounts** - Banking, healthcare, admin accounts
3. **Inform users** - Explain that biometric/PIN improves security

### For Authentication

1. **Match registration settings** - If registered with `required`, authenticate with `required`
2. **Allow `preferred` for convenience** - Users can choose faster auth if device supports it
3. **Document your policy** - Make clear what level of verification is expected

### Conditional Logic

{% code lineNumbers="true" %}
```php
// Require UV only for privileged operations
$isAdminAction = ($user->getRole() === 'ROLE_ADMIN');

$userVerification = $isAdminAction
    ? AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_REQUIRED
    : AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED;

$options = PublicKeyCredentialRequestOptions::create(
    challenge: random_bytes(32),
    userVerification: $userVerification
);
```
{% endcode %}

## Common Issues

### Issue: "User verification not supported"

**Cause**: User's authenticator doesn't support UV, but you set `required`.

**Solution**: Use `preferred` instead, or provide alternative authentication methods.

### Issue: Users complain about entering PIN repeatedly

**Cause**: Setting `required` for all operations, even low-risk ones.

**Solution**: Use `discouraged` or `preferred` for routine operations, save `required` for sensitive actions.

## Security Considerations

* ✅ **Higher assurance** - UV provides stronger identity verification than UP alone
* ✅ **Phishing resistant** - Even if an attacker steals the authenticator, they need the PIN/biometric
* ⚠️ **Not foolproof** - Biometric systems can have false positives; PINs can be observed
* ⚠️ **Device dependent** - Relying solely on UV limits compatible devices

## Constants Reference

The library provides convenient constants in the `AuthenticatorSelectionCriteria` class:

{% code lineNumbers="true" %}
```php
use Webauthn\AuthenticatorSelectionCriteria;

// For registration (attestation)
AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_REQUIRED
AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_PREFERRED
AuthenticatorSelectionCriteria::USER_VERIFICATION_REQUIREMENT_DISCOURAGED

// These same constants can be used for authentication (assertion)
// in PublicKeyCredentialRequestOptions
```
{% endcode %}

## See Also

* [Authenticators](authenticators.md) - Learn about different authenticator types
* [Authenticator Selection Criteria](../pure-php/advanced-behaviours/authenticator-selection-criteria.md) - Detailed configuration
* [Ceremonies](ceremonies.md) - How user verification fits into the authentication flow
* [WebAuthn Specification - User Verification](https://www.w3.org/TR/webauthn-2/#user-verification) - Official specification
