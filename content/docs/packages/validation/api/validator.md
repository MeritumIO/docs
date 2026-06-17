+++
title = "Validator"
description = "Interface for validating input against a schema"
weight = 1
+++

# Validator

`Meritum\Validation\Validator`

The primary interface for validation. Register `ValidationModule` and inject `Validator::class` into your services.

## Methods

```php
/**
 * @param array<string, array<int|string, mixed>> $schema
 * @param array<string, mixed>                    $input
 */
public function validate(array $schema, array $input): ValidationResult;
```

Validates `$input` against `$schema` and returns an immutable [`ValidationResult`](/docs/packages/validation/api/validation-result/). The schema maps field names to a list of rule names and parameters. See the [guide](/docs/packages/validation/) for schema syntax, presence rules, dot notation, and wildcard expansion.

## Example

```php
use Meritum\Validation\Validator;

final class RegisterUserAction
{
    public function __construct(private readonly Validator $validator) {}

    public function execute(array $input): void
    {
        $result = $this->validator->validate([
            'username' => ['required', 'string', 'lengthBetween' => [3, 32]],
            'email'    => ['required', 'email'],
            'password' => ['required', 'string', 'lengthMin' => 8],
            'role'     => ['optional', 'in' => ['user', 'admin']],
        ], $input);

        if (!$result->passed()) {
            throw new ValidationException($result->getErrors());
        }
    }
}
```
