# Register Additional Authenticators

In some circumstances, you may need to register a new authenticator for a user e.g. when adding a new authenticator or when an administrator acts as another user to replace a lost device.

It is possible to perform this ceremony programmatically.

{% hint style="success" %}
You can attach several authenticators to a user account. It is recommended in case of lost devices or if the user gets access on your application using multiple platforms \(smartphone, laptop…\).
{% endhint %}

## The Easy Way

The procedure is the same as [the one described in this page](../the-webauthn-server/the-easy-way/register-a-new-authentication.md), except that you don’t have to save the user entity again.

## The Hard Way

The procedure is the same as [the one described in this page](../the-webauthn-server/the-hard-way/authenticator-registration.md).

## The Symfony Way

{% hint style="info" %}
The following procedure is only available with the version 3.1.0 of the framework. For previous versions, please refer to the Hard Way above.
{% endhint %}

With a Symfony application, the fastest way for a user to register additional authenticators is to use the helper `Webauthn\Bundle\Service\AuthenticatorRegistrationHelper` provided by the bundle.

In the example below, we will create 2 routes: the first one to get the options, te second one to verify the authenticator response. These routes will return JSON responses, but you are free to use Twig templates or any of response type.

{% code title="src/Controller/AddAuthenticatorController.php" %}
```php
<?php

declare(strict_types=1);

namespace App\Controller;

use Assert\Assertion;
use Symfony\Bridge\PsrHttpMessage\HttpMessageFactoryInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Component\Serializer\SerializerInterface;
use Symfony\Component\Validator\Validator\ValidatorInterface;
use Throwable;
use Webauthn\AuthenticatorAttestationResponseValidator;
use Webauthn\Bundle\Security\Storage\SessionStorage;
use Webauthn\Bundle\Service\AuthenticatorRegistrationHelper;
use Webauthn\Bundle\Service\PublicKeyCredentialCreationOptionsFactory;
use Webauthn\PublicKeyCredentialLoader;
use Webauthn\PublicKeyCredentialSourceRepository;
use Webauthn\PublicKeyCredentialUserEntity;

class AddAuthenticatorController extends AbstractController
{
    /**
     * @var AuthenticatorRegistrationHelper
     */
    private $helper;

    public function __construct(HttpMessageFactoryInterface $httpMessageFactory, ValidatorInterface $validator, SerializerInterface $serializer, PublicKeyCredentialCreationOptionsFactory $publicKeyCredentialCreationOptionsFactory, PublicKeyCredentialSourceRepository $publicKeyCredentialSourceRepository, PublicKeyCredentialLoader $publicKeyCredentialLoader, AuthenticatorAttestationResponseValidator $authenticatorAttestationResponseValidator, SessionStorage $optionsStorage)
    {
        $this->helper = new AuthenticatorRegistrationHelper(
            $publicKeyCredentialCreationOptionsFactory,
            $serializer,
            $validator,
            $publicKeyCredentialSourceRepository,
            $publicKeyCredentialLoader,
            $authenticatorAttestationResponseValidator,
            $optionsStorage,
            $httpMessageFactory
        );
    }

    /**
     * @Route(name="add_authenticator_options", path="/add/device/options", schemes={"https"}, methods={"POST"})
     */
    public function getOptions(Request $request): Response
    {
        try {
            $userEntity = $this->getUser();
            Assertion::isInstanceOf($userEntity, PublicKeyCredentialUserEntity::class, 'Invalid user');
            $publicKeyCredentialCreationOptions = $this->helper->generateOptions($userEntity, $request);

            return $this->json($publicKeyCredentialCreationOptions);
        } catch (Throwable $e) {
            return $this->json([
                'status' => 'failed',
                'errorMessage' => 'Invalid request',
            ], 400);
        }

    }

    /**
     * @Route(name="add_authenticator", path="/add/device", schemes={"https"}, methods={"POST"})
     */
    public function addAuthenticator(Request $request): Response
    {
        try {
            $userEntity = $this->getUser();
            Assertion::isInstanceOf($userEntity, PublicKeyCredentialUserEntity::class, 'Invalid user');
            $this->helper ->validateResponse($userEntity, $request);
            return $this->json([
                'status' => 'ok',
                'errorMessage' => '',
            ]);
        } catch (Throwable $e) {
            return $this->json([
                'status' => 'failed',
                'errorMessage' => 'Invalid request',
            ], 400);
        }
    }
}
```
{% endcode %}

{% hint style="success" %}
If the current user is registering authenticators for another user \(admin\), the userEntity passed to the methods `generateOptions` and `validateResponse` must correspond to the target user.
{% endhint %}

{% hint style="info" %}
This controller may be directly integrated in the bundle. 
{% endhint %}

By default the options profile is `default`. You can change it setting the profile name as third argument of the method `generateOptions`.

```php
$publicKeyCredentialCreationOptions = $this->helper->generateOptions(
    $userEntity,
    $request,
    'profile1'
);
```

{% hint style="warning" %}
As the user shall be authenticated to register a new authenticator, you should protect these routes in the `security.yaml` file.
{% endhint %}

{% code title="config/packages/security.yaml" %}
```yaml
security:
    access_control:
        - { path: ^/add/device,  roles: IS_AUTHENTICATED_FULLY }
```
{% endcode %}

