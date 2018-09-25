# Upgrade Guide

## Upgrading to v4 from v3

We aim to document all the changes that could impact your theme, and there may only be a portion that are applicable to your theme.

### Composer

Update `lumberjack-core` to version 4.

```javascript
"rareloop/lumberjack-core": "^4.0"
```

### PHP Version

{% hint style="warning" %}
**Likelihood Of Impact: High**
{% endhint %}

Support for **PHP 7.0 has been dropped**, ensure you're running at least **PHP 7.1**.

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

The query builder is now part of the core, rather than an external package. If you were using the package, you will need to remove its service provider from your list of `providers` above.

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

### Container

{% hint style="warning" %}
**Likelihood Of Impact: Medium**
{% endhint %}

The `bind()` method on the `Application` container is no longer a singleton by default when the value \(2nd param\) is not a primitive or object instance.

When [binding a concrete implementation to an interface](container/using-the-container.md#set-concrete-implementations-for-interfaces), using singletons by default can create unexpected side affects as state is maintained across instances.

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

### `ServerRequest` class \(optional\)

{% hint style="warning" %}
**Likelihood Of Impact: Optional, but recommended**
{% endhint %}

If you're injecting an instance of the Diactoros `ServerRequest` class into a Controller, you can now switch this out for the following class if you want to benefit from some of the [new helper functions](the-basics/http-requests.md#usage):

```text
Rareloop\Lumberjack\Http\ServerRequest
```

For example:

```text
use Rareloop\Lumberjack\Http\ServerRequest;

class MyController
{
    public function show(ServerRequest $request)
    {
        $name = $request->input('name');
    }
}
```

{% hint style="info" %}
If you have enabled [global helpers](the-basics/helpers.md#request), you can use access the current `ServerRequest` instance using the `request()`helper instead of using dependency injection. For example:

```php
class MyController
{
    public function show()
    {
        $name = $request->input('name');
    }
}
```
{% endhint %}

Here's a quick overview of what the new `ServerRequest` object can do. _If you are using_ [_global helpers_](the-basics/helpers.md#request)_, you can replace `$request` with `request()` instead in the examples below:_

#### **Query Parameters**

```php
// Get all query parameters
$request->query();

// Get a specific query parameter
$request->query('name');

// Get a specific query parameter with a default
$request->query('name', 'Jane');
```

#### Input

```php
// Get all input params (from $_GET and $_POST)
$request->input();

// Get a specific input parameter
$request->input('name');

// Get a specific input parameter, with a default
$request->input('name', 'Jane');

// Check if the request has a key
$request->has('name');
```

### View Models

{% hint style="warning" %}
**Likelihood Of Impact: Medium**

This is a previously undocumented feature. If you are using ViewModels, this is a major change to how they work. However, if you are not using ViewModels you do not need to do anything.
{% endhint %}

View Models are simple classes that allow you to transform data that would otherwise be defined in your controller. This allows for better encapsulation of code and allows your code to be re-used across your controllers \(and even across themes\).

Head over to the new View Model documentation to learn more:

{% page-ref page="the-basics/view-models.md" %}

#### Upgrading existing ViewModels

The `ViewModel` base class no longer extends from `stdClass` and so can no longer have arbitrary properties set on it. 

We'd suggest upgrading your existing ViewModels to either use public methods or public properties. If your project has a large number of ViewModel's, the simplest change is to specifically name all properties in the class.

For example:

```php
// Lumberjack v3
use Rareloop\Lumberjack\ViewModel;

class MyViewModel extends ViewModel
{
    public function __construct($post)
    {
        $this->title = $post->title;
        $this->link = $post->link;
    }
}
```

To:

```php
// Lumberjack v4
use Rareloop\Lumberjack\ViewModel;

class MyViewModel extends ViewModel
{
    // Declare the class properties
    public $title;
    public $link;

    public function __construct($post)
    {
        $this->title = $post->title;
        $this->link = $post->link;
    }
}
```

### Binding of the Exception Handler

{% hint style="warning" %}
**Likelihood Of Impact: Very low**
{% endhint %}

In `bootstrap/app.php` you should change how the exception handler is bound to `Rareloop\Lumberjack\Exceptions\HandlerInterface`.

From:

```php
$app->bind(HandlerInterface::class, $app->make(Handler::class));
```

To:

```php
$app->singleton(HandlerInterface::class, Handler::class);
```

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

**If you're just using the Router Facade, you do not need to change anything.**

From:

```text
use Rareloop\Router\Router
```

To:

```text
Rareloop\Lumberjack\Http\Router
```

### PSR-15 Middleware

{% hint style="warning" %}
**Likelihood Of Impact: Very low**
{% endhint %}

The `http-interop/http-server-middleware` package has been deprecated in favour of the now official PSR-15 interfaces found in `psr/http-server-middleware`.

Make sure any middleware used now complies with the `Psr\Http\Server\MiddlewareInterface` interface.

