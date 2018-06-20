# Helpers

## Introduction

Lumberjack provides a handful of helpful functions that should make your life a little easier.

By default they are available as static methods on the `c` class, so that the global namespace isn't getting polluted. You can safely use these methods without fear of function names clashing.

You can tell Lumberjack to create global functions for you if you do not mind about adding these functions to the global namespace.

If you are building packages or plugins specifically for Lumberjack, you cannot rely on the global helper functions as the theme may not have made them available.

## Available Helpers

- app
- config
- view
- route
- redirect

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
\Rareloop\Lumberjack\Helpers::view('templates/posts.twig', $context, 200, $headers);

// Global function
view('templates/posts.twig', $context, 200, $headers);
```