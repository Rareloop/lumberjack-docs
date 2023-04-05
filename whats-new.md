# What's New

## What's new in v6.0

Lumberjack v6.0 comes with PHP 8.1 support. It also replaced the (deprecated) [tightenco/collect](https://packagist.org/packages/tightenco/collect) package with [illuminate/collections](https://packagist.org/packages/illuminate/collections) package.

### Features

Lumberjack will no longer give a 500 error for deprecation warnings, and instead will only "report" these to your error log.

Specifically, lumberjack will only report (and not render) the following error codes:

* `E_DEPRECATED` - [_Run-time notices. Enable this to receive warnings about code that will not work in future versions._](#user-content-fn-1)[^1]
* `E_USER_DEPRECATED` - _User-generated warning message. This is like an E\_DEPRECATED, except it is generated in PHP code by using the PHP function trigger\_error()._
* `E_USER_NOTICE` - _User-generated notice message. This is like an E\_NOTICE, except it is generated in PHP code by using the PHP function trigger\_error()._

If you need to change this behaviour, you can add a new config file `config/errors.php` in your theme, like so:

```php
<?php

return [
    'reportOnly' => [E_USER_ERROR],
];
```

## What's new in v4.3

This release of Lumberjack is jam packed full of goodies. We have also added a whole lot more documentation, so grab a cuppa and make yourself comfy while we take you through all the changes.

### Features

#### Middleware Aliases on routes and controllers

You can now create Middleware Aliases that can be used anywhere that middleware can normally be used, both in the Router and through Controllers.

If, for example, an Alias had been registered for an `AuthMiddleware` with the key `auth` like so:

```php
MiddlewareAliases::set('auth', function() {
    return new AuthMiddleware;
});
```

You can now use this in your route definition like this:

```php
Router::get(
    'route/uri', 
    '\App\Http\Controllers\TestController@testMethod'
)->middleware('auth');
```

{% content-ref url="the-basics/middleware.md" %}
[middleware.md](the-basics/middleware.md)
{% endcontent-ref %}

#### `logger()` helper

The `logger` helper can be used to write **debug** messages to your logs.

```php
\Rareloop\Lumberjack\Helpers::logger('Product added to basket', ['id' => $product->id]);

// Global function
logger('Product added to basket', ['id' => $product->id]);
```

If you need to access the logger class itself, to log different types of errors for example, you can use the `logger` function with no arguments. This will get you an instance of the PSR3 compliant logger that is bound to the container. By default Lumberjack uses `Monolog\Logger`.

```php
\Rareloop\Lumberjack\Helpers::logger()->warning('Example warning');

// Global function
logger()->warning('Example warning');
```

{% hint style="info" %}
Also, the `Logger` instance is now also bound to the PSR-3 interface `Psr\Log\LoggerInterface` in the Container.
{% endhint %}

{% content-ref url="the-basics/helpers.md" %}
[helpers.md](the-basics/helpers.md)
{% endcontent-ref %}

### Patches

The `E_USER_NOTICE` and `E_USER_DEPRECATED` errors are now whitelisted from the Exception Handler so that they aren’t fatal.

## What's new in v4.2

### Features

#### Query Builder

The Query Builder became macroable & also received one new useful method: `first()`.

You can now add your own functionality to the query builder by using a macro. In this example we are adding a custom `search` method:

```php
use Rareloop\Lumberjack\QueryBuilder;

// Add custom function
QueryBuilder::macro('search', function ($term) {
    $this->params['s'] = $term;
    
    return $this;
});

// Use the functionality
$query = new QueryBuilder();
$query->search('Elephant');

$posts = $query->first();
```

{% content-ref url="the-basics/query-builder.md" %}
[query-builder.md](the-basics/query-builder.md)
{% endcontent-ref %}

#### Define middleware in controllers

You can now apply Middleware on a Controller class too, either for use with the [Router](the-basics/routing.md) or as a [WordPress Controller](the-basics/wordpress-controllers.md). In order to do this your Controller must extend the `App\Http\Controllers\Controller` base class.

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

{% content-ref url="the-basics/middleware.md" %}
[middleware.md](the-basics/middleware.md)
{% endcontent-ref %}

#### Config has()

Lumberjack's configuration class now lets you check whether a config file contains a given item:

```php
if (Config::has('app.mySetting') {
    // ...
}
```

Note that the `has` method only checks whether the config item exists, regardless of its value.

If you set `app.mySetting` to an empty value such as `false` or `null`, `has('app.mySetting')` will return `true`.

{% content-ref url="getting-started/configuration.md" %}
[configuration.md](getting-started/configuration.md)
{% endcontent-ref %}

### Documentation

We have also added/revisited some of the documentation. We recommend checking these out:

* [View Models](the-basics/view-models.md) - New documentation
* [Middleware](the-basics/middleware.md) - New documentation
* [Collections](the-basics/collections.md) - New documentation

## What's new in v4.1

### Features

#### Extending Lumberjack with Macros

You can now extend core Lumberjack classes and add your own functionality without needing to rely on inheritance. Instead, you can add macros (custom functions) to the core classes themselves.

Here's an example macro, that adds a custom `acf()` method on `Rareloop\Lumberjack\Post`.

```php
use Rareloop\Lumberjack\Post;

// Add custom function
Post::macro('acf', function ($field) {
    return get_field($field, $this->id);
});

// Use the functionality
$post = new Post;
$value = $post->acf('custom_field_name');
```

The following classes are 'macroable':

* `Rareloop\Lumberjack\Post`
* `Rareloop\Router\Router`
* `Rareloop\Router\RouteGroup`
* `Rareloop\Router\Route`

## What's new in v4.0

### General

#### PHP Version

The first important thing to mention is that the minimum version of PHP has been bumped up to `7.1`. So make sure your server can handle this version.

#### Standardising the Container

Lumberjack uses a [dependency injection container](container/using-the-container.md) under the hood, which allows you to decouple your class dependencies by having the container inject dependencies when needed.

In v3, it wasn't clear when you were resolving a singleton or a new instance of a class from the container. It could have lead to some unusual & unexpected issues, so we decided to make things much clearer with v4 and standardise the behaviour.

Now, when you bind a **class name** into the container like so:

```php
$app->bind('key', MyClass::class);

// Or bind a concrete class to an interface
$app->bind(MyInterface::class, MyClass::class);
```

The container will **always resolve a new instance**. What does that mean? In short, it means state is not kept between resolves, you will always get a new instance of the class. Here's a couple of examples to illustrate the point:

```php
// Bind a class to the container
$app->bind('key', MyClass::class);

$value1 = $app->get('key');
$value2 = $app->get('key');

$value1 === $value2; // false
```

Here we are resolving `key` from the container twice. Each time it is resolved, the container will create a new instance of the `MyClass` class. If we were to modify `$value1` in any way, that change **would not** persist in `$value2`. For example:

```php
// Bind a class to the container
$app->bind('key', MyClass::class);

$value1 = $app->get('key');

// Modify $key1
$value1->name = 'Adam';

// Get 'key' from the container again
$value2 = $app->get('key');

// Throws an exception as name is not defined on $value2
$value2->name;
```

For the majority of the time, we feel like this is the better option. It means you are not accidentally coupling your application to the state of something in the container.

However, sometimes you do want that behaviour. When you do, you can simply use the `singleton` method to tell the container to resolve the same instance if there is one.

```php
$app->singleton('key', MyClass::class);

// Or bind a concrete class to an interface
$app->singleton(MyInterface::class, MyClass::class);
```

Now, when the container resolves the instance it will use the one that is already bound to the container. For example:

```php
// Bind a singleton to the container
$app->singleton('key', MyClass::class);

$value1 = $app->get('key');
$value2 = $app->get('key');

$value1 === $value2; // true
```

And when we modify the instance, its state will persist:

```php
// Bind a class to the container
$app->singleton('key', MyClass::class);
​
$value1 = $app->get('key');
​
// Modify $value1
$value1->name = 'Adam';
​
// Get 'key' from the container again
$value2 = $app->get('key');
​
// 'name' is available because $value2 is the same object as $value1
$value2->name; // 'Adam'
```

{% hint style="info" %}
It is important to note that if you bind **an object instance** to the container, you will always get that instance back. For example:

```php
$app->bind('key', new MyClass);

$value1 = $app->get('key');
$value1->name = 'Adam';

$value2 = $app->get('key');

// Both variables reference the same object
$value1 === $value2; // true

$value2->name; // 'Adam'
```
{% endhint %}

Head over to the "Using the Container" docs to learn more:

{% content-ref url="container/using-the-container.md" %}
[using-the-container.md](container/using-the-container.md)
{% endcontent-ref %}

### Features

#### New helper functions

To make your development lives easier, there are now some additional helper functions available. These are:

* `redirect()` - returns a `RedirectResponse`
* `back()` - returns a `RedirectResponse` which automatically redirects back to the previous URL
* `report($exception)` - tells the Exception Handler to report an exception. Useful if your theme needs to swallow an exception, but you still want to log the fact that it happened
* `request()` - returns the current `ServerRequest` object
* `session()` - can be used to interact with the session in various ways

Check out the Helpers documentation for more details:

{% content-ref url="the-basics/helpers.md" %}
[helpers.md](the-basics/helpers.md)
{% endcontent-ref %}

#### Query Builder

We've baked-in the [rareloop/lumberjack-querybuilder](https://github.com/Rareloop/lumberjack-querybuilder) package into the core. You now get an expressive, fluent and explicit way of querying data in WordPress out-of-the-box with Lumberjack. It can be used instead of [WP\_Query](https://codex.wordpress.org/Class\_Reference/WP\_Query) to query posts (of any type) and means you do not have to worry about "the loop".

{% content-ref url="the-basics/query-builder.md" %}
[query-builder.md](the-basics/query-builder.md)
{% endcontent-ref %}

#### Sessions

This is one of the bigger features added to v4. You can now manage sessions in a concise, expressive and headache-free way.

Let's dive straight into what sessions look like in Lumberjack. We'll be using the [global helper function](the-basics/helpers.md#session) `session()` for these examples; [_make sure you have enabled them_ ](the-basics/helpers.md#adding-global-helpers)_if you want to use it too._

```php
// Get a value, with a default value
$value = session('key', 'default');

// Get all values
$values = session()->all();

// Store a value
session()->put('key', 'value');

// Check if the session has a value
session()->has('key');

// Flash a value for 1 request
session()->flash('key', 'value');

// Remove a value
session()->forget('key');
```

Be sure to read the Sessions documentation for a more in-depth look:

{% content-ref url="the-basics/session.md" %}
[session.md](the-basics/session.md)
{% endcontent-ref %}

#### Interacting with the request

In v3, you could access the request by dependency injecting `Zend\Diactoros\ServerRequest`, like so:

```php
use Zend\Diactoros\ServerRequest;

class ExampleController
{
    public function handle(ServerRequest $request)
    {

    } 
}
```

Lumberjack now has its own `ServerRequest` class, making it much easier to work with the request. You can access it like so:

```php
use Rareloop\Lumberjack\Http\ServerRequest;

class ExampleController
{
    public function handle(ServerRequest $request)
    {

    } 
}
```

You can also use the `request()` helper to access the request from anywhere in your theme:

```php
use Rareloop\Lumberjack\Helpers;

$request = Helpers::request();

// Or if you have global helpers enabled:
$request = request();
```

**Overview of some available methods**

```php
// Get the current path
$request->path(); // e.g. /path

// Get the current URL
$request->url(); // e.g. http://test.com/path

// Get the current URL, with query parameters etc
$request->fullUrl(); // e.g. http://test.com/path?name=adam

// Get query params
$request->query();
$name = $request->query('name', 'default');

// Get all input (from $_GET and $_POST)
$input = $request->input();
$name = $request->input('name', 'default');

// Check the request has a specific key
$request->has('name');
```

You can read the HTTP Requests documentation for more information:

{% content-ref url="the-basics/http-requests.md" %}
[http-requests.md](the-basics/http-requests.md)
{% endcontent-ref %}

### Documentation

We have also added/revisited some of the documentation. We recommend checking these out:

* [Upgrade Guide](upgrade-guide.md) - How to upgrade to v4 from v3
* [HTTP Requests](the-basics/http-requests.md) - New feature!
* [Sessions](the-basics/session.md) - New feature!
* [Using the Container](container/using-the-container.md) - Revisited docs after the changes to the container's behaviour
* [Helpers](the-basics/helpers.md) - Added more helpers

[^1]: 
