# Lifecycle

## Introduction

Understanding how Lumberjack works and fits into WordPress's lifecycle will help demystify some of the 'magic'. It can also be important to know the order in which things happen so your application doesn't have unexpected behaviour.

## Starting at the beginning

As WordPress uses a front controller, all requests will go through the `index.php` file. This then bootstraps WordPress, runs your `functions.php`, finds which template to load in your theme and then renders some HTML.

Lumberjack is initialised within your `functions.php` file, meaning it bootstraps and runs before any pages in the [template hierarchy](http://pressupinc.com/blog/2013/08/understanding-wordpress-template-hierarchy/).

## Service Providers

Service providers are fundamental to bootstrapping the rest of the framework and all of its various parts. Within your theme's `config/app.php` file you have an item for `providers`. These are all the core components that make up Lumberjack.

```php
/**
 * List of providers to initialise during app boot
 */
'providers' => [
    Rareloop\Lumberjack\Providers\RouterServiceProvider::class,
    Rareloop\Lumberjack\Providers\WordPressControllersServiceProvider::class,
    Rareloop\Lumberjack\Providers\TimberServiceProvider::class,
    Rareloop\Lumberjack\Providers\ImageSizesServiceProvider::class,
    Rareloop\Lumberjack\Providers\CustomPostTypesServiceProvider::class,
    Rareloop\Lumberjack\Providers\MenusServiceProvider::class,
    Rareloop\Lumberjack\Providers\LogServiceProvider::class,
    Rareloop\Lumberjack\Providers\ThemeSupportServiceProvider::class,
    Rareloop\Lumberjack\Providers\LogServiceProvider::class,
],
```

By having them defined in your theme, you can remove or modify any of the core behaviour. Do this with caution though. This also means you can add your own service providers too and power up Lumberjack in your own way!

Lumberjack loops through this array twice. The first time it calls the `register()` method on all of the service providers. Then after they are all register it calls the `boot()` method on each one. You can **read more about service providers here** \*Todo\*.

## Processing The Request

The first service provider that runs is quite a fundamental one. The `RouterServiceProvider` has 2 important roles, which both trigger when WordPress had finished instantiating all plugins and the theme \(during the [`wp_loaded` action](https://codex.wordpress.org/Plugin_API/Action_Reference/wp_loaded)\).

First, it transforms the HTTP request into a PSR7 compliant `ServerRequest` object. **Read more about what that means here** \*Todo\*

Next, it checks to see if any custom routes \(from your theme's `routes.php` file\) match the current URL. If it find a match, the request is handled and a response is returned. Otherwise the request carries on and gets handled by WordPress normally later.

**That means any custom routes will always supersede any WordPress content with the same URL.**

### WordPress routes

Lumberjack doesn't interfere with how WordPress handles the request or how it decides which template to load \(via the template hierarchy\).

`WordPressControllersServiceProvider` does however tweak the behaviour around _how_ the template is included. It hooks into `template_include` to check if the file has a controller class with the correct name. If it finds one, it calls the `handle()` method. Otherwise that file is treated as per normal.  
This means that you don't have to use controllers if you don't want to.

