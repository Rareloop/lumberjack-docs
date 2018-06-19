# Container

## Introduction

Lumberjack features a PSR11 compatible container, powered by the popular open source [PHPDI](http://php-di.org/). If this is a new term for you checkout this [great intro](http://php-di.org/doc/understanding-di.html) and don't worry, you don't have to make use of it if you don't want to.

{% hint style="info" %}
Todo: flesh this out more and add more examples
{% endhint %}

## Accessing the container

In the default Lumberjack `functions.php` you'll find the following code:

```php
$app = new Application(__DIR__);
```

This creates the Application and the `$app` variable becomes your reference to the container.

## Setting entries in the container

To add something to the container you simply call `bind()`, e.g.:

```php
$app->bind('foo', 'bar');
```

## Retrieving entries from the container

To retrieve an entry from the container you can use `get()`, e.g.:

```php
$foo = $app->get('foo');
```

## Use the container to create an object

You can use the container to create an object from any class that your application can autoload using `make()`, e.g.:

```php
$comment = $app->make('\MyNamespace\Comment');
```

When creating an object using `make`, all the dependencies required by it's `__construct()` function will be automatically resolved from the container using type hinting. If your object requires additional parameters that can not resolved by type hinting you can pass them as a second param, e.g.:

```php
namespace MyNamespace;

class Comment
{
  public function __construct(ClassInContainer $resolvable, $param1, $param2) {}
}

...

$comment = $app->make('\MyNamespace\Comment', [
  'param1' => 123,
  'param2' => 'abc',
]);
```

## Set concrete implementations for interfaces

You can also tell the container what concrete class to use when resolving a certain type hinted interface. This allows you to write your app code against contracts and then use the container to switch in the correct implementation at runtime.

```php
namespace App;

interface PaymentGateway {}

class StripePaymentGateway implements PaymentGateway{}

// In your service provider
$app->bind('\App\PaymentGateway', '\App\StripePaymentGateway');

// In your application
$gateway = $app->make('\App\PaymentGateway');
// $gateway is instance of '\App\StripePaymentGateway'
```

## Dependency Injection

{% hint style="info" %}
TODO: If you would like to know more about this, please submit an issue
{% endhint %}

