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

