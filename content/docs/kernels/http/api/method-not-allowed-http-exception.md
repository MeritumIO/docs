+++
title = "MethodNotAllowedHttpException"
description = "Thrown when the HTTP method is not allowed for the requested route"
weight = 7
+++

# MethodNotAllowedHttpException

`Meritum\Http\Exception\MethodNotAllowedHttpException`

Extends [`HttpException`](/docs/kernels/http/api/http-exception/). Represents a `405 Method Not Allowed` response.

## Constructor

```php
public function __construct(
    ServerRequestInterface $request,
    string $message = '',
    array $allowedMethods = [],
    ?Throwable $previous = null
)
```

`$allowedMethods` is a `string[]` of HTTP methods that are valid for the route. Stored on the public `$allowedMethods` property.

```php
use Meritum\Http\Exception\MethodNotAllowedHttpException;

throw new MethodNotAllowedHttpException($request, '', ['GET', 'HEAD']);
```

| Property | Value |
|---|---|
| Status | `405` |
| Title | `Method Not Allowed` |
