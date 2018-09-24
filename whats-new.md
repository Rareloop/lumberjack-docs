# What's New

This release of Lumberjack is jam packed full of goodies. We have also added a whole lot more documentation, so grab a cuppa and make yourself comfy while we take you through all the changes.

## General

### PHP Version

The first important thing to mention is that the minimum version of PHP has been bumped up to `7.1`. So make sure your server can handle this version.

### Container

Lumberjack uses a dependency injection container under the hood, which allows you to decouple your class dependencies by having the container inject dependencies when needed.

In v3, it wasn't clear when you were resolving a singleton or a new instance of a class from the container. It could have lead to some unusual & unexpected issues, so we decided to make things much clearer with v4.

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

## Features

* Sessions
* Helpers

## Docs

* Helpers
* HTTP Requests
* View Models



