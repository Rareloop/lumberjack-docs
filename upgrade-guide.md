# Upgrade Guide

## Upgrading to v4 from v3

### PHP Version

{% hint style="warning" %}
**Likelihood Of Impact: High**
{% endhint %}

Support for PHP 7.0 has been dropped, ensure you're running at least PHP 7.1.

### Container

{% hint style="warning" %}
**Likelihood Of Impact: Medium**
{% endhint %}

The `bind()` method on the `Application` container is no longer a singleton by default when the value \(2nd param\) is not a primitive or object instance.

When binding a concrete implementation to an interface, being a singleton created unexpected side affects.

A new `singleton()` method has been provided to enable the previous behaviour. This enables the app developer to be more intentional about the behaviour they desire.

For example:

```php
// Singleton
$app->singleton(App\AppInterface::class, App\AppImplementation::class);
$object1 = $app->get(App\AppInterface::class);
$object2 = $app->get(App\AppInterface::class);

// The same object is resolved from the container
$object1 === $object2; // true

// Bind
$app->bind(App\AppInterface::class, App\AppImplementation::class);
$object1 = $app->get(App\AppInterface::class);
$object2 = $app->get(App\AppInterface::class);

// The container resolves new instances, so the objects are not the same
$object1 === $object2; // false
```

### Service Providers

{% hint style="warning" %}
**Likelihood Of Impact: Critical**
{% endhint %}

Add the following providers to `config/app.php`:

```php
'providers' => [
    ...
    Rareloop\Lumberjack\Providers\QueryBuilderServiceProvider::class,
    Rareloop\Lumberjack\Providers\SessionServiceProvider::class,
    Rareloop\Lumberjack\Providers\EncryptionServiceProvider::class,
],
```

### PSR-15 Middleware

{% hint style="warning" %}
**Likelihood Of Impact: Very low**
{% endhint %}

The `http-interop/http-server-middleware` package has been deprecated in favour of the now official PSR-15 interfaces found in `psr/http-server-middleware`.

Make sure any middleware used now complies with the `Psr\Http\Server\MiddlewareInterface` interface.

### Exception Handler

{% hint style="warning" %}
**Likelihood Of Impact: Critical**
{% endhint %}

The type hint on the `render()` function has changed to the PSR interface from the concrete Zend implementation.

Make the following change in `app/Exceptions/Handler.php`:

From:

```php

use Zend\Diactoros\ServerRequest;

public function render(ServerRequest $request, Exception $e) : ResponseInterface
{

}
```

To:

```php

use Psr\Http\Message\ServerRequestInterface;

public function render(ServerRequestInterface $request, Exception $e) : ResponseInterface
{

}
```

No changes should be required to your application logic as Zend subclasses will already comply with the new interface.

### `Helpers::app()` helper

{% hint style="warning" %}
**Likelihood Of Impact: Very low**
{% endhint %}

`Helpers::app()` \(and the `app()` global counterpart\) no longer use the `make()` method of the Application instance and now rely on `get()`. This provides much more consistent behaviour with other uses of the Container. If you still want to use the helpers to get `make()` behaviour you can change your code.

From:

```php
Helpers::app(MyClassName::class);
```

To:

```php
Helpers::app()->make(MyClassName::class);
```

### `Router` class namespace

{% hint style="warning" %}
**Likelihood Of Impact: Very low**
{% endhint %}

If you resolve an instance of the `Router` class from the container, you'll need to change the class reference.

If you're just using the Router Facade, you do not need to change anything.

From:

```text
use Rareloop\Router\Router
```

To:

```text
Rareloop\Lumberjack\Http\Router
```

### `ServerRequest` class \(optional\)

{% hint style="warning" %}
**Likelihood Of Impact: Very low**
{% endhint %}

If you're injecting an instance of the Diactoros `ServerRequest` class into a Controller, you can now switch this out for the following class if you want to benefit from some of the [new helper functions](the-basics/http-requests.md#usage):

```text
Rareloop\Lumberjack\Http\ServerRequest
```



