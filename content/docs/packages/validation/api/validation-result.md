+++
title = "ValidationResult"
description = "Immutable result returned by Validator::validate()"
weight = 2
+++

# ValidationResult

`Meritum\Validation\ValidationResult`

Immutable result object returned by `Validator::validate()`. Holds the pass/fail outcome and all error messages.

## Methods

```php
public function passed(): bool;
```

Returns `true` if no rules failed.

```php
/** @return array<string, string[]> */
public function getErrors(): array;
```

Returns all validation errors, keyed by field name. Each key maps to an array of error message strings — one per failed rule. Fields with no errors are absent from the returned array.

## Example

```php
$result = $validator->validate([
    'name'  => ['required', 'string', 'lengthMax' => 100],
    'email' => ['required', 'email'],
], $input);

if (!$result->passed()) {
    // e.g. ['email' => ['The email must be a valid email address']]
    $errors = $result->getErrors();
}
```
