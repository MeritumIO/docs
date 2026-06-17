+++
title = "HttpException"
description = "Base class for HTTP exceptions"
weight = 5
+++

# HttpException

`Meritum\Http\Exception\HttpException`

Base class for HTTP exceptions. Extends `RuntimeException` and implements [`HttpExceptionInterface`](/docs/kernels/http/api/http-exception-interface/). Defaults to a `500 Internal Server Error`. Extend it to create custom HTTP exceptions with specific status codes and titles.

## Constructor

```php
public function __construct(
    ServerRequestInterface $request,
    string $message = '',
    ?Throwable $previous = null
)
```

## Extending

```php
final class UnprocessableEntityException extends HttpException
{
    protected string $title = 'Unprocessable Entity';
    protected int $status = 422;
}
```

## Built-in subclasses

| Class | Status | Title |
|---|---|---|
| `NotFoundHttpException` | `404` | `Not Found` |
| `MethodNotAllowedHttpException` | `405` | `Method Not Allowed` |
