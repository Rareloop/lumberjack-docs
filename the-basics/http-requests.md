# HTTP Requests

## Accessing the Request Instance

To access the current Request object you can inject it into your Controller by using the `Rareloop\Lumberjack\Http\ServerRequest` type hint, e.g.

```php
use Rareloop\Lumberjack\Http\ServerRequest;

class MyController
{
    public function show(ServerRequest $request)
    {

    }
}
```

You can also use the `request()` [helper](helpers.md#adding-global-helpers) to access the request from anywhere in your theme:

```php
use Rareloop\Lumberjack\Helpers;
​
$request = Helpers::request();
​
// Or if you have global helpers enabled:
$request = request();
```

## Usage

### Get the method

```php
$request->getMethod(); // e.g. GET
$request->isMethod('GET'); // e.g. true
```

### Get the path

```php
$request->path(); // e.g. /path
```

### Get the URL

```php
$request->url(); // e.g. http://test.com/path
$request->fullUrl(); // e.g. http://test.com/path?foo=bar
```

### Get all query params

```php
$request->query();
```

### Get a specific query param

```php
$request->query('name');
$request->query('name', 'Jane'); // Defaults to "Jane" if not set
```

### Get all post params

```php
$request->post();
```

### Get a specific post param

```php
$request->post('name');
$request->post('name', 'Jane'); // Defaults to "Jane" if not set
```

### Get all input params

```php
$request->input();
```

### Get a specific input param

```php
$request->input('name');
$request->input('name', 'Jane'); // Defaults to "Jane" if not set
```

### Does the request have a specific input key?

```php
if ($request->has('name')) {
    // do something
}

if ($request->has(['name', 'age'])) {
    // do something if both 'name' and 'age' are present
}
```

