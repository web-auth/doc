# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the **documentation repository** for the WebAuthn Framework PHP library and Symfony bundle. It contains comprehensive GitBook-formatted documentation for implementing WebAuthn (strong authentication) in PHP and Symfony applications.

The actual PHP library code is maintained in a separate repository: https://github.com/web-auth/webauthn-framework

## Documentation Structure

The documentation is organized into the following sections:

### Core Sections
- **the-project/**: General project information, installation, browser support, and contributing guidelines
- **webauthn-in-a-nutshell/**: Conceptual overview of WebAuthn (authenticators, ceremonies, user verification, attestation, extensions)
- **prerequisites/**: Required setup before implementation (Relying Party, Credential Source Repository, User Entity Repository, JavaScript)

### Implementation Guides
- **pure-php/**: Using the core PHP library without frameworks (server setup, input loading/validation, registration, authentication)
- **symfony-bundle/**: Symfony-specific integration (bundle installation, Doctrine entities, firewall configuration)
- **symfony-ux/**: Symfony UX components for WebAuthn

### Additional Resources
- **migration/**: Version upgrade guides
- **.gitbook/**: GitBook configuration and assets

## File Organization

All documentation files are Markdown (.md) with YAML frontmatter for GitBook metadata. The `SUMMARY.md` file defines the table of contents structure for GitBook.

Key structural elements:
- Files use GitBook-specific syntax including `{% hint %}` blocks and `{% code %}` blocks
- Images are stored in `.gitbook/assets/`
- Naming convention: Classes, constants, and properties match the W3C WebAuthn specification exactly

## Documentation Concepts

### Two WebAuthn Ceremonies
1. **Attestation/Creation Ceremony**: Registering an authenticator to a user account
2. **Assertion/Request Ceremony**: Authenticating a user with their authenticator

### Key Components
- **Relying Party (RP)**: The web application requesting authentication
- **Credential Source Repository**: Storage for authenticator credentials
- **User Entity Repository**: User account information
- **Authenticator**: Hardware/software device for authentication (security keys, platform authenticators)

## Working with This Repository

### Editing Documentation
- Maintain GitBook-specific syntax (hints, code blocks with lineNumbers)
- Keep frontmatter YAML at the top of each file
- Follow the existing structure in SUMMARY.md when adding new pages
- Use snake_case or kebab-case for filenames
- Match the naming conventions from the W3C WebAuthn specification for technical terms

### GitBook Integration
This repository uses GitBook for publishing. Changes pushed to the repository are automatically synced to GitBook. The GitBook workspace is managed by Spomky-Labs.

### Contributing Guidelines (from contributing.md)
- Follow PSR-12 coding standards (for PHP library, not this repo)
- Use PSR-20 for time handling
- Target the correct branch for PRs:
  - Bug fixes: Use the first major release branch (e.g., 1.0.x, 2.0.x, 3.0.x)
  - New features: Use the last minor release branch
- Default branch is v5.0 (current version)

### Version Strategy
The project uses semantic versioning with dedicated branches for each minor version:
- Each minor version has its own branch (e.g., `1.1.x`, `1.2.x`, `5.0.x`)
- Default branch is set to the latest version

## Key Documentation Patterns

### PHP Code Examples
Code examples show PHP usage of the WebAuthn library classes such as:
- `Webauthn\PublicKeyCredentialRpEntity` - Relying Party configuration
- `Webauthn\AuthenticatorAssertionResponse` - Authentication responses
- `Webauthn\PublicKeyCredentialCreationOptions` - Registration options

### Hint Blocks
The documentation uses several hint types:
- `{% hint style="info" %}` - General information
- `{% hint style="warning" %}` - Important warnings
- `{% hint style="danger" %}` - Critical security information
- `{% hint style="success" %}` - Correct usage examples

## Recent Documentation Improvements (November 2025)

The following corrections have been made to improve documentation quality:

### Spelling and Grammar Fixes
- Fixed typos: "Firewal" → "Firewall", "Usenrame" → "Username", "pleaase" → "please"
- Corrected grammar: "the the" → "the", "is why" → "which is why"
- Fixed missing words: "Note that is mandatory" → "Note that it is mandatory"

### File Naming Improvements
- `migration/from-v3.x-to-v4.0-1.md` → `migration/from-v5.x-to-v6.0.md` (now matches content)
- `symfony-ux/integration-1.md` → `symfony-ux/user-registration.md` (descriptive name)
- `symfony-ux/integration-2.md` → `symfony-ux/additional-authenticators.md` (descriptive name)

### Fixed Internal Links
- Corrected link in `symfony-bundle/advanced-behaviors/register-authenticators.md` to point to correct section
- Fixed Stimulus controller usage: `stimulus_controller('signin')` → `stimulus_action('signin')` for buttons

### Code Example Corrections
- Fixed YAML syntax errors (missing quotes)
- Corrected Stimulus controller invocations in templates

## External References

- Main GitHub repository: https://github.com/web-auth/webauthn-framework
- W3C WebAuthn Spec: https://www.w3.org/TR/webauthn-2/
- FIDO Alliance: https://fidoalliance.org/
- Project sponsor: https://github.com/sponsors/Spomky
