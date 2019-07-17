# Facades

Lumberjack uses the [Blast Facades](https://github.com/phpthinktank/blast-facades) library.

## Creating a Facade

Facades provide a simple static API to an object that has been registered into the container. For example to setup a facade you would first use a Service Provider to register an instance of your class into the container:

```php
namespace Rareloop\Lumberjack\Providers;

use Monolog\Logger;
use Rareloop\Lumberjack\Application;

class LogServiceProvider extends ServiceProvider
{
    public function register()
    {
        // Create object instance and bind into container
        $this->app->bind('logger', new Logger('app'));
    }
}
```

Then create a Facade subclass and tell it which key to use to retrieve your class instance:

```php
namespace Rareloop\Lumberjack\Facades;

use Blast\Facades\AbstractFacade;

class Log extends AbstractFacade
{
    protected static function accessor()
    {
        return 'logger';
    }
}
```

## Available Facades

The following Facades are already registered within Lumberjack.

* [Config](facades.md#Config)
* [Log](facades.md#Log)
* [Router](facades.md#Router)
* [Session](facades.md#Session)

### Config

The `config` facade allows you to get and set values from your config.

```php
$value = Config::get('app.environment');
```

Config is also available as a [Helper](https://docs.lumberjack.rareloop.com/the-basics/helpers#config)

### Log

The `log` facade allows you to use the [PSR-3](https://www.php-fig.org/psr/psr-3/) Logger [Monolog](https://github.com/Seldaek/monolog).

```php
$value = Log::warning('oops');
```

### Router

The `router` facade allows you to define named routes.

See [Creating Routes](https://docs.lumberjack.rareloop.com/the-basics/routing#creating-routes) for more information.

### Session

The `session` facade allows you to retrieve and store data in the current session.

```php
$name = Session::get('name');
```

Session is also available as a [Helper](https://docs.lumberjack.rareloop.com/the-basics/helpers#session)
