+++
title = "NotFoundHttpException"
description = "Thrown when a requested resource does not exist"
weight = 6
+++

# NotFoundHttpException

`Meritum\Http\Exception\NotFoundHttpException`

Extends [`HttpException`](/docs/kernels/http/api/http-exception/). Represents a `404 Not Found` response.

```php
use Meritum\Http\Exception\NotFoundHttpException;

throw new NotFoundHttpException($request);
throw new NotFoundHttpException($request, 'User not found');
```

| Property | Value |
|---|---|
| Status | `404` |
| Title | `Not Found` |
