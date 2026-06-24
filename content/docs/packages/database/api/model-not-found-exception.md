+++
title = "ModelNotFoundException"
description = "Thrown by findOrFail and firstOrFail when no model is found"
weight = 6
+++

# ModelNotFoundException

`Meritum\Database\Exception\ModelNotFoundException`

Extends `RuntimeException`. Thrown by `Repository::findOrFail()` and `Repository::firstOrFail()` when no matching model exists. The message is formatted as `"{ModelName} was not found"`.

## Usage

```php
use Meritum\Database\Exception\ModelNotFoundException;

try {
    $user = $userRepository->findOrFail($id);
} catch (ModelNotFoundException) {
    // return a 404 response
}
```

When using `meritum/http-exception-handler`, register a `TranslationHandler` to map `ModelNotFoundException` to an `HttpNotFoundException` so the error response carries the correct 404 status code and error code automatically.
