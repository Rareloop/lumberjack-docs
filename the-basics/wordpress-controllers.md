# WordPress Controllers

## Introduction

In the files (i.e. controllers) that WordPress uses when it matches a route (e.g. `page.php`, `single.php`), you can now use a class rather than writing procedural code.

```php
// page-home.php

/*
 * Template Name: Home Template
 */

namespace App;

use Timber\Timber;
use Rareloop\Lumberjack\Post;
use Rareloop\Lumberjack\Http\Responses\TimberResponse;

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
* The class name must be an UpperCamelCase version of the filename with the word `Controller` on the end (without spaces, dashes and underscores). If the controller name is not correctly Lumberjack will not throw any errors - instead you will just get a blank page.

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

## Password protected pages

{% hint style="info" %}
Since: v6.1.0
{% endhint %}

WordPress supports password protection for pages across your site. This is also [supported by Timber](https://timber.github.io/docs/v1/guides/wp-integration/#password-protected-posts) but requires you to implement it in each of your site's templates to take effect universally.

To make this simpler to implement, Lumberjack adds a middleware to handle this across all page templates. When a request is made for a page that requires a password it will intercept the call and attempt to render the `single-password.twig` file instead.

It is recommended that you implement the Twig file in line with what Timber suggests:

```twig
{% raw %}
{% extends "base.twig" %}

{% block content %}
    {{ function('get_the_password_form') }}
{% endblock %}
{% endraw %}
```

{% hint style="info" %}
**Note:** If a `single-password.twig` file is **not found,** the middleware will fall back gracefully to the default behaviour, which is to **ignore the password functionality and render the page as normal**.
{% endhint %}

### Changing the Twig file used for password protected pages

If you would like to change the name of the Twig file that is used for password protection you can do so using the `lumberjack/password_protect_template` filter:

```php
add_filter('lumberjack/password_protect_template', function() {
    return 'my-password-template.twig';
});
```
