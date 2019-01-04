# Helpers

## Introduction

Lumberjack provides a handful of helpful functions that should make your life a little easier.

By default they are available as static methods on the `\Rareloop\Lumberjack\Helpers` class, so that the global namespace isn't getting polluted. You can safely use these methods without fear of function names clashing.

You can tell Lumberjack to create global functions for you if you do not mind about adding these functions to the global namespace.

If you are building packages or plugins specifically for Lumberjack, you cannot rely on the global helper functions as the theme may not have made them available.

## Adding Global Helpers

In order to use the global helper functions, all you need to do it tell composer to autoload `vendor/rareloop/lumberjack-core/src/functions.php`.

Add the following to your top-level `composer.json` file:

```javascript
"autoload": {
    "files": [
        "vendor/rareloop/lumberjack-core/src/functions.php"
    ]
}
```

Then, tell composer to regenerate its list of autoloaded files by running `composer dump-autoload`.

## Available Helpers

* [app](helpers.md#app)
* [config](helpers.md#config)
* [view](helpers.md#view)
* [route](helpers.md#route)
* [redirect](helpers.md#redirect)
* [session](helpers.md#session)
* [request](helpers.md#request)
* [back](helpers.md#back)
* [report](helpers.md#report)

### app

The `app` helper returns the current reference to the container.

```php
$app = \Rareloop\Lumberjack\Helpers::app();

// Global function
$app = app();
```

You can resolve objects from the container by passing in the class name or reference into `app()`.

```php
$gateway = \Rareloop\Lumberjack\Helpers::app(\App\PaymentGateway::class);

// Global function
$gateway = app(\App\PaymentGateway::class);
```

### config

The `config` helper allows you to get values from your config.

```php
$value = \Rareloop\Lumberjack\Helpers::config('app.environment');

// Global function
$value = config('app.environment');
```

By passing in an array to the `config` helper you can set config values.

```php
\Rareloop\Lumberjack\Helpers::config(['app.logs.enabled' => false]);

// Global function
config(['app.logs.enabled' => false]);
```

### view

The `view` helper returns a new `Rareloop\Lumberjack\Http\Responses\TimberResponse`.

```php
return \Rareloop\Lumberjack\Helpers::view('templates/posts.twig', $context, 200, $headers);

// Global function
return view('templates/posts.twig', $context, 200, $headers);
```

### route

The `route` helper generates a URL from a named route.

```php
$url = \Rareloop\Lumberjack\Helpers::route('posts.index');

// Global function
$url = route('posts.index');
```

If the route requires parameters you can be pass an associative array as a second parameter:

```php
$url = \Rareloop\Lumberjack\Helpers::route('posts.show', ['id' => 123]);

// Global function
$url = route('posts.show', ['id' => 123]);
```

### redirect

The `redirect` helper returns a new `Zend\Diactoros\Response\RedirectResponse`, which redirects the user to a given URL.

```php
$url = \Rareloop\Lumberjack\Helpers::redirect('/auth/login', 200, $headers);

// Global function
$url = redirect('/auth/login', 200, $headers);
```

### session

You can use the `session` helper to retrieve and store data in the current session. Passing in 1 \(string\) argument will get the value of that item from the session. Passing in an array of key/value pairs will add each pair to the session.

```php
// Get a value from the session
$name = \Rareloop\Lumberjack\Helpers::session('name');

// Get a value from the session, with a default
$name = \Rareloop\Lumberjack\Helpers::session('name', 'default');

// Set a value to the session
\Rareloop\Lumberjack\Helpers::session(['key' => 'value']);

// Call any method on a session
\Rareloop\Lumberjack\Helpers::session()->forget('key');
```

And using the global function instead:

```php
// Get a value from the session
$name = session('name');

// Get a value from the session, with a default
$name = session('name', 'default');

// Store a value to the session
session(['key' => 'value']);

// Call any method on a session
session()->forget('key');
```

### request

The `request` helper returns the current `ServerRequest` object, which means you get access to [all these available methods](http-requests.md#usage).

```php
$request = \Rareloop\Lumberjack\Helpers::request();

// Global function
$request = request();
```

For example, to get the current url:

```php
$currentUrl = request()->url(); // e.g. http://test.com/path
```

{% page-ref page="http-requests.md" %}

### back

Returns a RedirectResponse, which will redirect the user to the previous URL.

```php
return \Rareloop\Lumberjack\Helpers::back();

// Global function
return back();
```

If you need to pass any data back to the previous page, you can flash items to the session using `with()`:

```php
// Chain 'with' multiple times
return \Rareloop\Lumberjack\Helpers::back()
    ->with('key', 'value')
    ->with('foo', 'bar');

// Or use an array with key/value pairs
return \Rareloop\Lumberjack\Helpers::back()->with([
    'key' => 'value,
    'foo' => 'bar',
]);

// Global function
return back()
    ->with('key', 'value')
    ->with('foo', 'bar');

// Or use an array with key/value pairs
return back()->with([
    'key' => 'value,
    'foo' => 'bar',
]);
```

### report

Calls the `report` method on the Exception Handler, to ensure that an exception is reported.

```php
\Rareloop\Lumberjack\Helpers::report($exception);

// Global function
report($exception);
```

This is particularly useful if your theme needs to _swallow_ any exceptions so they do not break the site, but you still wish to have the error logged. For example:

```php
try {
    // This may throw an exception...
    $comment = Comment::add($data);

    return JsonResponse($comment, 200);
} catch (Exception $exception) {
    // Report the exception
    report($exception);

    // Swallow the exception, and instead return a response
    return JsonResponse([], 400);
}
```

"Swallowing" here simply means that the exception is unable to bubble all the way up to the Exception Handler where it normally gets reported.

