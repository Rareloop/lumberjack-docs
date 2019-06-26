# Routing

## Introduction

Sometimes you may want to create a page on your website but not need/want it editable in WordPress. That's when the custom Lumberjack router comes into play. It can also be used to make AJAX endpoints.

{% hint style="warning" %}
If you set up a custom route that has the same URL as a WordPress page, the router takes priority.
{% endhint %}

{% hint style="info" %}
Route closures and controller functions are automatically dependency injected from the container.
{% endhint %}

## Creating Routes

Routing is handled by using the `Rareloop\Lumberjack\Facades\Router` Facade. The convention is to create your Routes in the `routes.php` file at the base of your theme.

Typically, you only need to allow one HTTP verb for a route \(e.g. `POST` or `GET`\). To create a route, use the HTTP verb as the method name. The first parameter is the URI and the second is the code you wish to execute when that route is matched.

```php
Router::get('test/route', function () {});
Router::post('test/route', function () {});
Router::put('test/route', function () {});
Router::patch('test/route', function () {});
Router::delete('test/route', function () {});
Router::options('test/route', function () {});
```

{% hint style="info" %}
WordPress doesn't know anything about custom routes, so you may need to manually handle things like the page meta \(e.g. title\). Generally it is best to use WordPress where you can and use custom routes for anything non-standard.

Some good candidates for a custom route could include:

* An AJAX endpoint
* An endpoint to POST a form to \(which could send an email\)
* A custom e-commerce checkout workflow \(e.g. basket, checkout, confirmation pages\)
{% endhint %}

### Setting the page title for custom routes

If you need to set the page title for your custom route, you can manually call the `wp_title` filter. For example:

```php
namespace App\Http\Controllers;

class TestController
{
    public function __construct()
    {
        add_filter('wp_title', function ($title) {
            return 'My Custom Title';
        });
    }
    
    public function show()
    {
        return 'Hello World';
    }
}
```

If your controller has multiple methods, then you could do something like this:

```php
namespace App\Http\Controllers;

class TestController
{
    protected $pageTitle;

    public function __construct()
    {
        add_filter('wp_title', function ($title) {
            if (!empty($this->pageTitle)) {
                return $this->pageTitle;
            }

            return $title;
        });
    }
    
    protected function setPageTitle(string $title) {
        $this->pageTitle = $title;
    }
    
    public function basket()
    {
        $this->setPageTitle('Basket');
        
        return 'Basket...';
    }
    
    public function checkout()
    {
        $this->setPageTitle('Checkout');
        
        return 'Checkout...';
    }
}
```



## Route Parameters

Parameters can be defined on routes using the `{keyName}` syntax. When a route that contains parameters is matched, those parameters are available as injectable parameters in your callback/controller. 

**The name of the route parameter and the controller parameter must be the same.**

```php
Router::get('posts/{id}', function($id) {

});
```

As the parameters are injected by name, it doesn't matter which order you have the parameters in your callback:

```php
// /posts/123/comments/1
Router::get('posts/{postId}/comments/{commentId}', function($commentId, $postId) {
    echo $commendId; // 1
    echo $postId; // 123
});
```

Or controller:

```php
// routes.php
Router::get('posts/{postId}/comments/{commentId}', 'TestController@show');

// app/Http/Controllers/TestController.php
namespace App\Http\Controllers;

class TestController
{
    public function show($postId, $commentId)
    {
        return 'Hello World';
    }
}
```

### Parameter Constraints

By default, all parameters will match against all non `/` characters. You can make the match more specific by supplying a regular expression:

```php
Router::get('posts/{id}', function () {})->where('id', '[0-9]+');

// Will match /posts/123
// Won't match /posts/abc
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

The string takes the format `{name of class}@{name of method}`. The Router will be default look for classes in the `App\Controllers` namespace, so you don't need to include this prefix in the Controller String.

```php
// routes.php
Router::get('route/uri', 'TestController@show');

// app/Http/Controllers/TestController.php
namespace App\Http\Controllers;

class TestController
{
    public function show()
    {
        return 'Hello World';
    }
}
```

If you want to reference a Controller in another namespace you'll need to append the complete namespace to the classname:

```php
// routes.php
Router::get('route/uri', '\My\Namespace\TestController@show');

// My/Namespace/TestController.php
namespace My\Namespace;

class TestController
{
    public function show()
    {
        return 'Hello World';
    }
}
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

## Extending the Router

The Lumberjack `Router` class can be extended with custom functionality at runtime \(the class is "macroable"\). The following example adds an `redirect` method to the `Router` class that can be used to easily declare redirect routes, without needing to handle the response logic yourself:

```php
use Rareloop\Lumberjack\Facades\Router;

// Add the custom functionality
Router::macro('redirect', function ($inputUrl, $outputUrl) {
    $this->get($inputUrl, function () use ($outputUrl) {
        return new RedirectResponse($outputUrl);
    });
});

// Use the custom functionality
Router::redirect('/old/url', '/new/url');
```

It is also possible to extend the `Route` class too, this can be useful for encapsulating common route behaviour in a more fluent API.

```php
use Rareloop\Lumberjack\Facades\Router;
use Rareloop\Router\Route;

// Add the custom functionality
Route::macro('adminOnly', function () {
    $this->middleware(new App\AdminMiddlewareOne);
    $this->middleware(new App\AdminMiddlewareTwo);
    $this->middleware(new App\AdminMiddlewareThree);
    
    return $this;
});

// Use the custom functionality
Router::get('route/uri', 'AdminController@action')->adminOnly();
```

{% hint style="warning" %}
Remember to `return $this` in your custom `Route` macro extensions so that the API remains chainable.
{% endhint %}

