+++
title = "ModelNotFoundException"
description = "Thrown by findOrFail when no model matches the given primary key"
weight = 6
+++

# ModelNotFoundException

`Meritum\Database\Exception\ModelNotFoundException`

Extends `RuntimeException`. Thrown by `Repository::findOrFail()` when no model with the requested primary key exists.

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
