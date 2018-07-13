# Routing

The Lumberjack Router is based on the standalone [Rareloop Router](https://github.com/Rareloop/router) but utilises a Facade to make setup and access simpler.

Sometimes you want to create a page on your website but do not want it editable in WordPress. That's when this router comes into play. It can also be used to make AJAX endpoints.

If you set up a custom route that has the same URL as a WordPress page, the router takes priority.

Note: Route closures and controller functions are automatically dependency injected from the container.

## Creating Routes

Typically, you only need to allow one HTTP verb for a route \(e.g. `POST` or `GET`\). To create a route, use the HTTP verb as the method name. The first parameter is the URI and the second is the code you wish to execute when that route is matched.

```php
Router::get('test/route', function () {});
Router::post('test/route', function () {});
Router::put('test/route', function () {});
Router::patch('test/route', function () {});
Router::delete('test/route', function () {});
Router::options('test/route', function () {});
```

## Route Parameters

Parameters can be defined on routes using the `{keyName}` syntax. When a route matches that contains parameters, an instance of the `RouteParams` object is passed to the action.

```php
Router::get('posts/{id}', function(RouteParams $params) {
    return $params->id;
});
```

## Named Routes

Routes can be named so that their URL can be generated programatically:

```php
Router::get('posts/all', function () {})->name('posts.index');

$url = Router::url('posts.index');
```

If the route requires parameters you can be pass an associative array as a second parameter:

```php
Router::get('posts/{id}', function () {})->name('posts.show');

$url = Router::url('posts.show', ['id' => 123]);
```

## Route Groups

It is common to group similar routes behind a common prefix. This can be achieved using Route Groups:

```php
Router::group('prefix', function ($group) {
    $group->get('route1', function () {}); // `/prefix/route1`
    $group->get('route2', function () {}); // `/prefix/route2`
});
```

## Route Controllers

If you'd rather use a class to group related route actions together you can pass a Controller String instead of a closure.

The string takes the format `{name of class}@{name of method}`. It is important that you use the complete namespace with the class name.

```php
// app/Http/Controllers/TestController.php
namespace App\Http\Controllers;

class TestController
{
    public function show()
    {
        return 'Hello World';
    }
}

// routes.php
Router::get('route/uri', '\App\Http\TestController@show');
```

### **Map**

If you need match a URL for multiple HTTP methods, you can use the `map` method and pass in an array of HTTP methods to match.

```php
use Rareloop\Lumberjack\Facades\Router;

// Creates a route that matches the uri `/posts/list` both GET
// and POST requests.
Router::map(['GET', 'POST'], 'posts/list', function () {
    return 'Hello World';
});
```

`map()` takes 3 parameters:

* `methods` \(array\): list of matching HTTP methods, valid values:
  * `GET`
  * `POST`
  * `PUT`
  * `PATCH`
  * `DELETE`
  * `OPTIONS`
* `uri` \(string\): The URI to match against
* `action` \(function\|string\): Either a closure or a Controller string

## Middleware

PSR-15/7 Middleware can be added to both routes and groups on the route.

### **Adding Middleware to a route**

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

### **Adding Middleware to a group**

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

