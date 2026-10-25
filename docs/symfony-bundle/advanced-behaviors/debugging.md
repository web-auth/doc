# Debugging

If you have troubles during the development of your application or if you want to keep track of every critical/error messages in production, you can use a [PSR-3 compatible logger](https://www.php-fig.org/psr/psr-3/).

```yaml title="config/packages/webauthn.yaml" linenums="1"
webauthn:
    logger: App\Service\MyPsr3Logger
```
