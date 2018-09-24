# HTTP Responses

## Introduction

All of your WordPress and Router controllers should return a PSR7 compliant response to be sent back to the user's browser. Lumberjack provides a number of Response types which should cover the majority of cases.

{% hint style="info" %}
If your Controller returns a string it will be automatically converted to an `HtmlResponse` object with a `200` status code.
{% endhint %}

## Available Response Objects

### Timber Response

The most common use-case will be to render a Twig view with some associated data. Using Timber on it's own, your code would look like this:

```php
use Timber\Timber;

...

return Timber::render('home.twig', $context);
```

In Lumberjack, you should take advantage of the `Rareloop\Lumberjack\Http\Responses\TimberResponse` object to achieve the same thing:

```php
use Rareloop\Lumberjack\Http\Responses\TimberResponse;

...

return new TimberResponse('home.twig', $context);
```

#### Context Flattening

When passing context data to TimberResponse, any objects that implement the `Rareloop\Lumberjack\Contracts\Arrayable` contract will automatically be flattened to a standard PHP array. This means that it is safe to use objects such as `Collection` and `ViewModel` in your data without it causing issues with Twig.

### Redirect Response

Redirecting to a different URL can be done by returning an instance of `Rareloop\Lumberjack\Http\Responses\RedirectResponse`.

```php
use Rareloop\Lumberjack\Http\Responses\RedirectResponse;

...

return new RedirectResponse('/another/page');
```

#### Adding Flash Data

If you want to redirect to a URL and also flash some data to the session, you can use the `with()` method.

```text
return new RedirectResponse('/another/page')
    ->with('error', 'Something went wrong');
```

### Diactoros Responses

Lumberjack also includes the fantastic [Zend Diactoros](https://github.com/zendframework/zend-diactoros) package which provides additional PSR7 compliant Response Objects.

```php
// Check out their documentation for further details and examples:
// https://zendframework.github.io/zend-diactoros/
$response = new Zend\Diactoros\Response\TextResponse('Hello world!');
$response = new Zend\Diactoros\Response\HtmlResponse($htmlContent);
$response = new Zend\Diactoros\Response\XmlResponse($xml);
$response = new Zend\Diactoros\Response\JsonResponse($data);
$response = new Zend\Diactoros\Response\EmptyResponse(); // Basic 204 response:
```

## Adding Status Code & Headers

One of the benefits of using Response Objects is that they make it easier to control the HTTP status code & headers.

All the Response Objects bundled with Lumberjack let you set both the status code and headers in their constructor.

```php
use Zend\Diactoros\Response\JsonResponse;

return new JsonResponse($data, 422, ['X-Total-Validation-Errors' => 2]);
```

Or if you prefer, you can use the `withStatus` and `withHeader` methods instead.

```php
use Zend\Diactoros\Response\JsonResponse;

return (new JsonResponse($data, 422))
    ->withStatus(422)
    ->withHeader('X-Total-Validation-Errors', 2);
```

## Responsable Objects

In addition to supporting PSR7 compliant responses, Controllers can also return an object that implements the `Rareloop\Router\Responsable` interface. These objects provide a `toResponse()` method that will return an instance of a PSR7 Response.

```php
// app/Http/Controllers/TestController.php
namespace App\Http\Controllers;

use App\Exceptions\TestException;

class TestController
{
    public function show()
    {
        return new TestException('Hello World');
    }
}

// app/Exceptions/TestException.php
namespace App\Exceptions;

use Rareloop\Router\Responsable;
use Psr\Http\Message\RequestInterface;
use Psr\Http\Message\ResponseInterface;
use Zend\Diactoros\Response\JsonResponse;

class TestException extends \Exception implements Responsable
{
    protected $reason;

    public function __construct($reason)
    {
        $this->reason = $reason;
        parent::__construct();
    }

    public function toResponse(RequestInterface $request) : ResponseInterface
    {
        return new JsonResponse([
            'error' => $this->reason,
        ], 400);
    }
}
```
