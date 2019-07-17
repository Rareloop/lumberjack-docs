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

Lumberjack comes with a handful of useful Facades. Below you can see which class the Facade references and what it is bound to the container under.

| Facade | Class Reference | Container binding |
| :--- | :--- | :--- |
| Config | `Monolog\Logger` | `config` |
| Log | `Rareloop\Lumberjack\Config` | `logger` |
| Router | `Rareloop\Lumberjack\Http\Route` | `router` |
| Session | `Rareloop\Lumberjack\Session\SessionManager` | `session` |

### Example usage

All of Lumberjack's Facades live under the `Rareloop\Lumberjack\Facades` namespace, and can be used like so:

#### Config

The `Config` facade allows you to get and set values from your config.

```php
use Rareloop\Lumberjack\Facades\Config;

Config::set('app.debug', false);

$value = Config::get('app.debug');
```

Config is also available as a [Helper](https://docs.lumberjack.rareloop.com/the-basics/helpers#config)

#### Log

The `Log` facade allows you to use the [PSR-3](https://www.php-fig.org/psr/psr-3/) Logger [Monolog](https://github.com/Seldaek/monolog).

```php
use Rareloop\Lumberjack\Facades\Log;

$value = Log::warning('Oops! Something went wrong');
```

#### Router

The `Router` facade allows you to create custom routes or get the URLs for your routes.

```php
use Rareloop\Lumberjack\Facades\Router;

Router::get('posts/all', function () {})->name('posts.index');

$url = Router::url('posts.index');
```

See [Creating Routes](https://docs.lumberjack.rareloop.com/the-basics/routing#creating-routes) for more information.

#### Session

The `Session` facade allows you to retrieve and store data in the current session.

```php
use Rareloop\Lumberjack\Facades\Session;

Session::put('name', 'Chris');

$name = Session::get('name');
```

Session is also available as a [Helper](https://docs.lumberjack.rareloop.com/the-basics/helpers#session)
