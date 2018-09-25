# What's New

This release of Lumberjack is jam packed full of goodies. We have also added a whole lot more documentation, so grab a cuppa and make yourself comfy while we take you through all the changes.

## General

### PHP Version

The first important thing to mention is that the minimum version of PHP has been bumped up to `7.1`. So make sure your server can handle this version.

### Standardising the Container

Lumberjack uses a [dependency injection container](container/using-the-container.md) under the hood, which allows you to decouple your class dependencies by having the container inject dependencies when needed.

In v3, it wasn't clear when you were resolving a singleton or a new instance of a class from the container. It could have lead to some unusual & unexpected issues, so we decided to make things much clearer with v4 and standardise the behaviour.

Now, when you bind a class name into the container like so:

```php
$app->bind('foo', Foo::class);

// Or use a fully qualified class name
$app->bind(FooInterface::class, Foo::class);
```

The container will **always resolve a new instance**. What does that mean? In short, it means state is not  kept between resolves, you will always get a new instance of the class. Here's a couple of examples to illustrate the point:

```php
// Bind a class to the container
$app->bind('foo', Foo::class);

$foo1 = $app->get('foo');
$foo2 = $app->get('foo');

$foo1 === $foo2; // false
```

Here we are resolving `foo` from the container twice. Each time it is resolved, the container will create a new instance of the `Foo` class. If we were to modify `$foo1` in any way, that change would not persist in `$foo2`. For example:

```php
// Bind a class to the container
$app->bind('foo', Foo::class);

$foo1 = $app->get('foo');

// Modify $foo1
$foo1->bar = true;

// Get 'foo' from the container again
$foo2 = $app->get('foo');

// Throws an exception as bar is not defined on $foo2
$foo2->bar;
```

For the majority of the time, we feel like this is the better option. It means you are not accidentally coupling your application to the state of something in the container.

However, sometimes you do want that behaviour. When you do, you can simply use the `singleton` method to tell the container to resolve the same instance if there is one.

```php
$app->singleton('foo', Foo::class);

// Or use a fully qualified class name
$app->singleton(FooInterface::class, Foo::class);
```

Now, when the container resolves the instance it will use the one that is already bound to the container. For example:

```php
// Bind a singleton to the container
$app->singleton('foo', Foo::class);

$foo1 = $app->get('foo');
$foo2 = $app->get('foo');

$foo1 === $foo2; // true
```

And when we modify the instance, its state will persist:

```php
// Bind a class to the container
$app->singleton('foo', Foo::class);
​
$foo1 = $app->get('foo');
​
// Modify $foo1
$foo1->bar = true;
​
// Get 'foo' from the container again
$foo2 = $app->get('foo');
​
// 'bar' is available because $foo2 is the same object as $foo1
$foo2->bar; // true
```

{% hint style="info" %}
It is important to note that if you bind **an object instance** to the container, you will always get that instance back. For example:

```php
$app->bind('foo', new Foo);

$foo1 = $app->get('foo');
$foo1->bar = true;

$foo2 = $app->get('foo');

// Both variables reference the same object
$foo1 === $foo2; // true

$foo2->bar; // true
```
{% endhint %}

[Head over to the "Using the Container" docs](container/using-the-container.md) to learn more.

## Features

### New helper functions

To make your development lives easier, there are now some additional helper functions available. These are:

* `redirect()` - returns a `RedirectResponse`
* `back()` - returns a `RedirectResponse` which automatically redirects back to the previous URL
* `report($exception)` - tells the Exception Handler to report an exception. Useful if your theme needs to swallow an exception, but you still want to log the fact that it happened
* `request()` - returns the current `ServerRequest` object
* `session()` - can be used to interact with the session in various ways

[Check out the Helpers documentation](the-basics/helpers.md) for more details.

### Sessions

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

Be sure to [read the Sessions documentation](the-basics/session.md) for a more in-depth look.

### Interacting with the request

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

#### Overview of some available methods

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

You can [read the HTTP Requests documentation](the-basics/http-requests.md) for more information.

## Documentation

We have also added/revisited some of the documentation. We recommend checking these out:

* [Upgrade Guide](upgrade-guide.md) - How to upgrade to v4 from v3
* [HTTP Requests](the-basics/http-requests.md) - New feature!
* [Sessions](the-basics/session.md) - New feature!
* [View Models](the-basics/view-models.md) - Added docs around existing feature
* [Using the Container](container/using-the-container.md) - Revisited docs after the changes to the container's behaviour
* [Helpers](the-basics/helpers.md) - Added more helpers



