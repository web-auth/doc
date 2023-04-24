---
description: Authenticator details and how to manage them
---

# Credential Source Repository

After the registration of an authenticator, you will get a Public Key Credential Source object. It contains all the credential data needed to perform user authentication and much more.

Each Credential Source is managed using the Public Key Credential Source Repository.

The library does not provide any concrete implementation. It is up to you to create it depending on your application constraints. This only constraint is that your repository class must implement the interface `Webauthn\PublicKeyCredentialSourceRepository`.

The `PublicKeyCredentialSourceRepository` interface requires the following methods to be implemented:

* `public function findOneByCredentialId(string $publicKeyCredentialId): ?PublicKeyCredentialSource;` : thie method retreive a key source object from the credential ID.
* `public function findAllForUserEntity(PublicKeyCredentialUserEntity $publicKeyCredentialUserEntity): array;`: this method lists all key sources associated to the user entity
* `public function saveCredentialSource(PublicKeyCredentialSource $publicKeyCredentialSource): void;`: this method saves the key source in your storage (files, database...)
