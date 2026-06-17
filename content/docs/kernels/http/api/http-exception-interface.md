+++
title = "HttpExceptionInterface"
description = "Contract for HTTP exceptions that carry a status code and title"
weight = 4
+++

# HttpExceptionInterface

`Meritum\Http\Exception\HttpExceptionInterface`

Extends `Throwable`. Implemented by `HttpException`, `NotFoundHttpException`, and `MethodNotAllowedHttpException`. Check for this interface in an exception handler to extract the HTTP status code and title without matching specific exception classes.

## Methods

```php
public function getRequest(): ServerRequestInterface;
```

Returns the request that was being handled when the exception was thrown.

```php
public function getTitle(): string;
```

Returns a human-readable title for the error (e.g. `'Not Found'`, `'Internal Server Error'`).

```php
public function getStatusCode(): int;
```

Returns the HTTP status code (e.g. `404`, `405`, `500`).

## Example

```php
use Meritum\Http\Exception\HttpExceptionInterface;

if ($e instanceof HttpExceptionInterface) {
    return new JsonResponse(
        ['error' => $e->getTitle()],
        $e->getStatusCode()
    );
}
```
