# Service Providers

## Introduction

When Lumberjack bootstraps it creates an Application Container, which is responsible for resolving concrete classes that have been **bound** to the container in one way or another. This binding happens via Service Providers.

A Service Provider needs to be registered with Lumberjack. This can be done in `config/app.php` under the `providers` array.

Let's look at how to create a new Service Provider and register it to the container.

## Creating Service Providers

Service Providers can live anywhere within the `app/` directory, or even as packages that you bring in via composer. Let's create a new file within `app/Providers` called `PaymentGatewayProvider.php`.

This file must extend `Rareloop\Lumberjack\Providers\ServiceProvider`. This gives you access to the container via `$this->app`. There are 2 methods that Lumberjack looks for on a Service Providers: `register()` and `boot()`.

### Register

The `register` method should only be used to bind things to the container. You cannot rely on Lumberjack registering service providers in any order, so you should always assume no other provider had been registered.

You should not attempt to do anything other than binding things to the container in \`register\`. Do not try and add WordPress filters/actions, register routes, configure WordPress etc here. This is to prevent you from accidentally using a service provider which has not been loaded yet.

```php
namespace App\Providers;

use Rareloop\Lumberjack\Providers\ServiceProvider;

class PaymentGatewayProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->set('\App\PaymentGateway', '\App\StripePaymentGateway');
    }
}
```

Now, whenever we need to use a payment gateway in our application we can resolve it from the container. This, in theory, makes it really easy to swap out Stripe with another payment gateway.

#### Binding to the container

There are a number of different ways to bind things to the container. Head over to ['Using the Container'](./) for more information.

### Boot

Once all service providers have been registered, Lumberjack then attempts to call the `boot` method on each one. This means that you have access to everything that has been bound to the container and can access it using [dependency injection ](./#dependency-injection)on the `boot` method.

```text
namespace App\Providers;

use Rareloop\Lumberjack\Providers\ServiceProvider;

/**
 * Add Option Pages to WP using the config, using ACF
 */
class OptionPagesProvider extends ServiceProvider
{
    // Dependency inject Config from the container
    public function boot(Config $config)
    {
        $optionPages = $config->get('option-pages');

        if (!is_array($optionPages)) {
            return;
         }

         foreach ($optionPages as $optionPage) {
             acf_add_options_page($optionPage);
         }
    }
}
```

### Register Service Providers with Lumberjack

Once you have your service provider ready to go, you need to add the name of the class to the `providers` array in `config/app.php`.

```php
return [
    'providers' => [
        // ...

        App\Providers\TwitterProvider::class,
    ],
];
```

