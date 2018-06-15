# HTTP Responses



## Introduction

All of your WordPress and Router controllers should return PSR7 compliant response. The most common use-case is rendering a `twig` view and passing in some context. Using Timber it would look like this:

```php
use Timber\Timber;

return Timber::render('home', $context);
```

In Lumberjack, you should take advantage of the `Rareloop\Lumberjack\Http\Responses\TimberResponse` object to achieve the same thing:

```php
use Rareloop\Lumberjack\Http\Responses\TimberResponse;

return new TimberResponse('home', $context);
```

The benefit to using these reponses is that they let you controll the HTTP status code & headers.

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

## Available Responses

As we make use of a great 3rd party package: [Zend Diactoros](https://github.com/zendframework/zend-diactoros), Lumberjack includes an array of handy responses out-of-the-box.

```php
$response = new Rareloop\Lumberjack\Http\Responses\TimberResponse('home', $context);

// Check out their documentation for further details and examples:
// https://zendframework.github.io/zend-diactoros/
$response = new Zend\Diactoros\Response\TextResponse('Hello world!');
$response = new Zend\Diactoros\Response\HtmlResponse($htmlContent);
$response = new Zend\Diactoros\Response\XmlResponse($xml);
$response = new Zend\Diactoros\Response\JsonResponse($data);
$response = new Zend\Diactoros\Response\EmptyResponse(); // Basic 204 response:
$response = new Zend\Diactoros\Response\RedirectResponse('/user/login');
```

