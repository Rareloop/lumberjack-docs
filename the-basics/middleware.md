# Middleware

Lumberjack supports PSR-15/7 Middleware. Middleware can be added via the [Router](routing.md) to individual routes or route groups or via [WordPress Controllers](wordpress-controllers.md).

## **Adding Middleware to a route**

At it's simplest, adding Middleware to a route can be done by passing an object to the `middleware()` function:

```php
$middleware = new AddHeaderMiddleware('X-Key1', 'abc');

Router::get('route/uri', '\MyNamespace\TestController@testMethod')->middleware($middleware);
```

Multiple middleware can be added by passing more params to the `middleware()` function:

```php
$header = new AddHeaderMiddleware('X-Key1', 'abc');
$auth = new AuthMiddleware();

Router::get('route/uri', '\MyNamespace\TestController@testMethod')->middleware($header, $auth);
```

Or alternatively, you can also pass an array of middleware:

```php
$header = new AddHeaderMiddleware('X-Key1', 'abc');
$auth = new AuthMiddleware();

Router::get('route/uri', '\MyNamespace\TestController@testMethod')->middleware([$header, $auth]);
```

## **Adding Middleware to a group**

Middleware can also be added to a group. To do so you need to pass an array as the first parameter of the `group()`function instead of a string.

```php
$header = new AddHeaderMiddleware('X-Key1', 'abc');

Router::group(['prefix' => 'my-prefix', 'middleware' => $header]), function ($group) {
    $group->get('route1', function () {}); // GET `/my-prefix/route1`
    $group->get('route2', function () {}); // GET `/my-prefix/route2`
});
```

You can also pass an array of middleware if you need more than one:

```php
$header = new AddHeaderMiddleware('X-Key1', 'abc');
$auth = new AuthMiddleware();

Router::group(['prefix' => 'my-prefix', 'middleware' => [$header, $auth]]), function ($group) {
    $group->get('route1', function () {}); // GET `/my-prefix/route1`
    $group->:get('route2', function () {}); // GET `/my-prefix/route2`
});
```

## **Defining Middleware on Controllers**

You can also apply Middleware on a Controller class too, either for use with the [Router](routing.md) or as a [WordPress Controller](wordpress-controllers.md). In order to do this your Controller must extend the `App\Http\Controllers\Controller` base class.

Middleware is added by calling the `middleware()` function in your Controller's `__constructor()`.

```php
use App\Http\Controllers\Controller;

class MyController extends Controller
{
    public function __construct()
    {
        // Add one at a time
        $this->middleware(new AddHeaderMiddleware('X-Key1', 'abc'));
        $this->middleware(new AuthMiddleware());

        // Add multiple with one method call
        $this->middleware([
            new AddHeaderMiddleware('X-Key1', 'abc',
            new AuthMiddleware(),
        ]);
    }
}
```

By default all Middleware added via a Controller will affect all methods on that class. To limit what methods Middleware applies to you can use `only()` and `except()`:

```php
use App\Http\Controllers\Controller;

class MyController extends Controller
{
    public function __construct()
    {
        // Only apply to `send()` method
        $this->middleware(new AddHeaderMiddleware('X-Key1', 'abc'))->only('send');

        // Apply to all methods except `show()` method
        $this->middleware(new AuthMiddleware())->except('show');

        // Multiple methods can be provided in an array to both methods
        $this->middleware(new AuthMiddleware())->except(['send', 'show']);
    }
}
```

## Middleware Aliases

{% hint style="info" %}
Available in `v4.3.0` and above
{% endhint %}

Creating new instances of middleware in your routes or controllers can have a couple of negative outcomes:

1. The objects are instantiated and take up memory whether they're used or not
2. Your route definitions can become less readable

These issues can both be addressed using Middleware Aliases. Instead of creating instances in your code, you register a string alias that can be used instead.

### Registering a Middleware Alias

To register an alias you can use the `MiddlewareAliases` facade. It is recommended that you use this in the `boot()` method of the `AppServiceProvider` class.

There are 3 ways to define your middleware alias:

* A closure factory _\(recommended - always creates a new instance when used\)_
* A class name _\(always resolves a new instance from the Container when used\)_
* A previously instantiated object _\(you don't get the benefits of lazy loading\)_

{% hint style="info" %}
_See "_[_Setting entries in the container_](https://app.gitbook.com/@rareloop/s/lumberjack/~/edit/drafts/-Lk9PktE5_xnzl2Zs2j1/container/using-the-container#setting-entries-in-the-container)_" for more information about the differences between these. While only the class name uses the container, principally they behave in the same way with regards to lazy-loading and object instantiation_.
{% endhint %}

#### Using a closure factory \(recommended\)

It is recommended that the alias is registered using a closure that acts as a factory, like so:

```php
namespace App\Providers;

use Rareloop\Lumberjack\Facades\MiddlewareAliases;

class AppServiceProvider
{
    function boot()
    {
        // Create from closure factory
        MiddlewareAliases::set('auth', function() {
            return new AuthMiddleware;
        });
    }
}
```

#### Using a class name

```php
// Create from class name
MiddlewareAliases::set('cors', \My\Middleware\Cors::class);

// Or
MiddlewareAliases::set('cors', '\My\Middleware\Cors');
```

#### Using a previously instantiated object

When using this method, you do not get the benefits of lazy loading. i.e. The object will be created whether or not it is needed for the current request.

```php
// Create from existing object
MiddlewareAliases::set('addheader', new \My\Middleware\AddHeader());
```

### Using a Middleware Alias

Middleware Aliases can be used anywhere middleware can normally be used, both in the Router and through Controllers.

If, for example, an Alias had been registered for an `AuthMiddleware` with the key `auth` like so:

```php
MiddlewareAliases::set('auth', function() {
    return new AuthMiddleware;
});
```

You could use this in your route definition like this:

```php
Router::get(
    'route/uri', 
    '\App\Http\Controllers\TestController@testMethod'
)->middleware('auth');
```

### Middleware Alias Parameters

You can register Middleware Aliases which accept parameters. It is advised if you're doing this to use the Closure Factory registration technique.

```php
// Register in AppServiceProvider::boot()
MiddlewareAliases::set('addheader', function($key, $value) {
    return new AddHeader($key, $value);
});

// Use in routes.php
Router::get(
    'hello-world', 
    'MyController@hello'
)->middleware('addheader:X-Key:HeaderValue');
```

In this example, `$key` will have the value `X-Key` and `$value` will have the value of `HeaderValue`.

