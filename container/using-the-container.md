# Using the Container

## Introduction

{% hint style="info" %}
Lumberjack features a PSR11 compatible container, powered by the popular open source [PHPDI](http://php-di.org/). If this is a new term for you checkout this [great intro](http://php-di.org/doc/understanding-di.html) and don't worry, you don't have to make use of it if you don't want to.

Having a deeper understanding of Lumberjack's dependency injection container will help you build maintainable, scalable and robust themes.
{% endhint %}

There are many ways in which you can interact with the container. Primarily it's used for dependency injection. You can either inject classes that Lumberjack has bound, or bind something yourself. This is a great way of managing class dependencies within your theme.

**The following examples will be using the** `app()` **global helper. If you do not have it enabled, you can use the** `Helpers` **class. For example:**

```php
app()->bind('key', 'value');
```

Would become:

```php
use Rareloop\Lumberjack\Helpers;
Helpers::app()->bind('key', 'value');
```

## Accessing the container

In the default Lumberjack `functions.php` you'll find the following code:

```php
$app = new Application(__DIR__);
```

This creates the Application and the `$app` variable becomes your reference to the container.

There are a couple of ways you can access the container throughout your application. Let's take a look at them.

#### Within Service Providers

The container can be accessed within a service provider by referencing `$this->app`.

```php
namespace App\Providers;

use Rareloop\Lumberjack\Providers\ServiceProvider;
use App\PaymentGateway;
use App\StripePaymentGateway;

class PaymentGatewayServiceProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->bind(PaymentGateway::class, StripePaymentGateway::class);
    }
}
```

{% hint style="warning" %}
You should only ever bind to the container within a service provider's `register()` method.
{% endhint %}

#### Everywhere else

You can access the container from anywhere in your theme by using the `app()` helper.

```php
use Rareloop\Lumberjack\Helpers;
$app = Helpers::app();
```

[Check out the "Helpers" documentation](../the-basics/helpers.md#app) for more information on using this helper.

## Resolving entries from the container

To resolve an entry from the container you can use `get()`.

```php
$value = app()->get('key');

// Or, using the helper's convenient shorthand
$value = app('key');
```

You can use the container to create an object from any class that your application can autoload using `get()`, for example:

```php
use \MyNamespace\Comment;

$comment = app->get(Comment::class);

// Or using the helper's convenient shorthand:
$comment = app(Comment::class);
```

When creating an object using `get`, all the dependencies required by it's `__construct()` function will be automatically resolved from the container using type hinting.

```php
namespace App;

class Comment
{
    public function __construct(ClassInContainer $resolvable)
    {
        $this->resolvable = $resolvable;
    }
}

...
use App\Comment;

$comment = app(Comment::class);
```

#### Make

If your object requires additional parameters that can not resolved by type hinting then you should use the `make` method, and pass them as a second param.

{% hint style="warning" %}
`make()` will always create a new instance and ignore any singleton binding.
{% endhint %}

For example:

```php
namespace App;

class Comment
{
    public function __construct(ClassInContainer $resolvable, $param1, $param2) {}
}

...

use App\Comment;

$comment = app()->make(Comment::class, [
    'param1' => 123,
    'param2' => 'abc',
]);
```

## Setting entries in the container

To add something to the container you simply call `bind()`. In this example, `value` is bound to the container under `key`.

```php
app()->bind('key', 'value');
```

You can bind pretty much anything you like to the container. Let's take a look at some other examples. Understanding the difference in behaviour is vital in using the container effectively.

### Objects

```php
use App\Comment;

app()->bind('comment', new Comment);
```

Whenever you resolve `comment` from the container, the same `Comment` object is returned. This is important, as it means **state** **is maintained automatically**.

```php
use App\Comment;

app()->bind('comment', new Comment);

$commentA = app('comment'); // resolves the 'Comment' object

// Update the comment
$commentA->author = 'Adam';

$commentB = app('comment'); // resolves the exact same 'Comment' object

$commentB->author; // 'Adam'
```

### Closures

Binding objects is useful, however it requires you to create an object before binding. This isn't always ideal, as it means that object has to be instantiated before being bound even if it is never used.

You can _lazy-load_ dependencies, where they are only created at the point they are needed, by passing in a closure to the container:

```php
use App\Comment;

app()->bind('comment', function () {
    return new Comment;
});
```

In this example, the `Comment` object is never created as nothing is trying to resolve `comment` from the container.

Now lets look at what happens when we try and resolve comment from the container:

```php
use App\Comment;

app()->bind('comment', function () {
    return new Comment;
});

$commentA = app('comment'); // Calls the closure and creates a new Comment object

// Update the comment
$commentA->author = 'Adam';

$commentB = app('comment'); // Calls the closure and creates a new Comment object

$commentB->author; // Throws an error, undefined property!
```

Here you can see that `$commentB->author` throws an error. **This is because closures will always create a new instance.**

If you need to persist state, you should bind using `singleton`**:**

```php
use App\Comment;

app()->singleton('comment', function () {
    return new Comment;
});
```

Now, if we revisit our example above `$commentB` will now be the same object as `$commentA`, and we won't get any errors.

```php
use App\Comment;

app()->singleton('comment', function () {
    return new Comment;
});

$commentA = app('comment'); // Calls the closure and creates a new Comment object

// Update the comment
$commentA->author = 'Adam';

$commentB = app('comment'); // resolves the exact same 'Comment' object

$commentB->author; // 'Adam'
```

### **Class names**

If you would like to resolve a class \(like our example `Comment` class\), you can use the class name when binding:

```php
use App\Comment;

app()->bind('comment', Comment::class); // Comment::class simply outputs to '\App\Comment'
```

By using a class name, the container behaves exactly the same as closures. **The container will only create the class when it gets used & it will resolve a new instance of the class every time**. For example:

```php
use App\Comment;

app()->bind('comment', Comment::class);

$commentA = app('comment'); // Creates a new instance of Comment

// Update the comment
$commentA->author = 'Adam';

$commentB = app('comment'); // Creates a new instance of Comment

$commentB->author; // Throws an error, undefined property!
```

However, you also get the added benefit of being able to inject dependencies into your classes `__construct()` method. See ["Retrieving entries from the container"](using-the-container.md#retrieving-entries-from-the-container) for more information on this.

Singletons work in the same way with class names as they do with closures. When you bind using `singleton()`, the same class instance is always resolved, and therefore its state is maintained.

```php
use App\Comment;

app()->singleton('comment', Comment::class);

$commentA = app('comment'); // Creates a new instance of Comment

// Update the comment
$commentA->author = 'Adam';

$commentB = app('comment'); // resolves the exact same instance of Comment

$commentB->author; // 'Adam'
```

### Set concrete implementations for interfaces

You can also tell the container what concrete class to use when resolving a certain type hinted interface. This allows you to write your app code against contracts and then use the container to switch in the correct implementation at runtime.

Lets walk through a quick example. First, lets create an interface \(or contract\) which states that payment gateways should be able to charge.

```php
namespace App;

interface PaymentGateway
{
    public function charge($amount);
}
```

Any implementation of this contract has to have a `charge` method that accepts an `$amount`. Lets add a Stripe implementation of the interface:

```php
namespace App;

class StripePaymentGateway implements PaymentGateway
{
    public function charge($amount)
    {
        // Charge using stripe...
    }
}
```

Within our application, we want to ask the container for `PaymentGateway`. This is because our application doesn't care which implementation to use, it just wants to be able to charge people. We can tell the container to resolve `StripePaymentGateway` whenever we ask for `PaymentGateway`, like so:

```php
app()->bind(PaymentGateway::class, StripePaymentGateway::class);

app(PaymentGateway::class); // resolves an instance of StripePaymentGateway
```

The same applies when dependency injecting:

```php
class MyController
{
    public function __construct(PaymentGateway $paymentGateway)
    {
        // $paymentGateway will be an instance of StripePaymentGateway
        // We can safely call 'charge' because our Stripe implementation has to adhere to the interface/contract
        $paymentGateway->charge(100);
    }
}
```

## Dependency Injection

Rather than manually resolving something from the container, you can type hint dependencies in the constructor of classes that are resolved by Lumberjack's container. These include: **controllers** and **service providers**.

If we have the following bound to the container:

```php
use App\MyClass;

app()->bind(MyClass::class, new MyClass);
```

You can type hint the `MyClass` class in a controller's constructor and the container will resolve it for you automatically:

```php
class MyController
{
    public function __construct(MyClass $myClass)
    {

    }
}
```

Generally, this is how most of your objects should be resolved from the container.

