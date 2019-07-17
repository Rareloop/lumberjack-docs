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

## Existing Facades

Lumberjack comes with a handful of useful Facades. Below you can see which class the Facade references and what it is bound to the container under.

| Facade | Class Reference | Container binding |
| :--- | :--- | :--- |
| Config | `Monolog\Logger` | `config` |
| Log | `Rareloop\Lumberjack\Config` | `logger` |
| Router | `Rareloop\Lumberjack\Http\Route` | `router` |
| Session | `Rareloop\Lumberjack\Session\SessionManager` | `session` |

### Example usage

All of Lumberjack's Facades live under the `Rareloop\Lumberjack\Facades` namespace, and can be used like so:

```php
use Rareloop\Lumberjack\Facades\Config;

Config::get('app.environment');
```

