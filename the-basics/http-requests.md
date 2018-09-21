# HTTP Requests

A Request object can be injected into your controllers, e.g.

```
use Rareloop\Lumberjack\Http\ServerRequest;

class MyController
{
    public function show(ServerRequest $request)
    {
    
    }
}
```

## Usage

### Get the path

```php
$request->path();
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
