+++
title = "FieldReferencingRuleInterface"
description = "Rule contract for comparing a field against another field in the input"
weight = 5
+++

# FieldReferencingRuleInterface

`Meritum\Validation\FieldReferencingRuleInterface`

Extends [`RuleInterface`](/docs/packages/validation/api/rule-interface/). Rules that implement this interface receive a field name as their schema parameter and have the target field's value resolved from the input before `validate()` is called.

`sameAs` and `differentFrom` implement this interface.

## Methods

```php
/**
 * @param array<string, mixed> $input
 * @return mixed[]
 */
public function resolveParams(string $attribute, array $input): array;
```

Called by the engine before `validate()`. `$attribute` is the target field name taken from the schema (e.g. `'password'` for `'sameAs' => 'password'`). Return the array of params to be spread into `validate()` and `message()`.

Typically this returns the resolved value alongside the field name for use in error messages:

```php
public function resolveParams(string $attribute, array $input): array
{
    return [$input[$attribute] ?? new Missing(), $attribute];
}
```

The engine validates that the schema param exists and is a string before calling `resolveParams()`. If it is missing or non-string, an `InvalidArgumentException` is thrown.

## Example

```php
use Meritum\Validation\FieldReferencingRuleInterface;
use Meritum\Validation\Missing;

final class GreaterThanFieldRule implements FieldReferencingRuleInterface
{
    public function name(): string
    {
        return 'greaterThanField';
    }

    public function resolveParams(string $attribute, array $input): array
    {
        return [$input[$attribute] ?? new Missing(), $attribute];
    }

    public function validate(mixed $value, mixed ...$params): bool
    {
        return is_numeric($value) && is_numeric($params[0]) && $value > $params[0];
    }

    public function message(string $attribute, mixed ...$params): string
    {
        return "The {$attribute} must be greater than {$params[1]}";
    }
}
```

Usage:

```php
'end_date' => ['required', 'date', 'greaterThanField' => 'start_date'],
```
