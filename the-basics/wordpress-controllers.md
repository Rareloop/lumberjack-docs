# WordPress Controllers

## Introduction

In the files \(i.e. controllers\) that WordPress uses when it matches a route \(e.g. `page.php`, `single.php`\), you can now use a class rather than writing procedural code.

```php
// page-home.php

/*
 * Template Name: Home Template
 */

namespace App;

use Timber\Timber;
use Rareloop\Lumberjack\Post;
use App\Http\Responses\TimberResponse;

class PageHomeController
{
    public function handle()
    {
        $context = Timber::get_context();

        $context['post'] = new Post;

        return new TimberResponse('home', $context);
    }
}
```

**The name of the controller is important:**

* It should be under the namespace `App`.
* The class name must be an UpperCamelCase version of the filename with the word `Controller` on the end \(without spaces, dashes and underscores\). If the controller name is not correctly Lumberjack will not throw any errors - instead you will just get a blank page.

The `handle` method will automatically be called on your controller.

## Changing the naming convention

If you wish to change how Lumberjack looks for controller class names, you can hook into the `lumberjack_controller_name` filter.

```php
add_filter('lumberjack_controller_name', function ($controllerName) {
    // e.g. Look for 'HomeController' instead of 'PageHomeController'
    return str_replace('Page', '', $controllerName);
});
```

Or if you wish to change the namespace you can use the `lumberjack_controller_namespace` filter.

```php
add_filter('lumberjack_controller_namespace', function ($controllerName) {
    // e.g. Look for 'MyApp\PageHomeController' instead of 'App\PageHomeController'
    return 'MyApp\\'
});
```

## Controller for the 404 page

In WordPress, you have a `404.php` file. `PHP` Classes cannot start with a number so following the usual naming convention will not work here.

Instead Lumberjack will look for a controller called `Error404Controller`

