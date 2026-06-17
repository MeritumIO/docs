+++
title = "RuleInterface"
description = "Contract for all validation rules"
weight = 3
+++

# RuleInterface

`Meritum\Validation\RuleInterface`

The base contract for all validation rules. Register implementations tagged with `'validation.rules'` to make them available in schemas.

## Methods

```php
public function name(): string;
```

Returns the rule's identifier as used in schemas (e.g. `'required'`, `'email'`, `'lengthMin'`). Must be unique within the registered rule set. Registering a rule with the same name as a built-in rule replaces it.

```php
public function validate(mixed $value, mixed ...$params): bool;
```

Returns `true` if `$value` passes the rule. `$value` may be a `Missing` sentinel if the field was absent from the input — rules must handle this case explicitly. `$params` are the values provided in the schema (e.g. `18` for `'min' => 18`).

```php
public function message(string $attribute, mixed ...$params): string;
```

Returns the error message for a failed validation. `$attribute` is the field name as it appears in the schema.

## Example

```php
use Meritum\Validation\RuleInterface;

final class SlugRule implements RuleInterface
{
    public function name(): string
    {
        return 'slug';
    }

    public function validate(mixed $value, mixed ...$params): bool
    {
        return is_string($value) && (bool) preg_match('/^[a-z0-9]+(?:-[a-z0-9]+)*$/', $value);
    }

    public function message(string $attribute, mixed ...$params): string
    {
        return "The {$attribute} must be a valid slug";
    }
}
```

See also [`StoppableRuleInterface`](/docs/packages/validation/api/stoppable-rule-interface/) and [`FieldReferencingRuleInterface`](/docs/packages/validation/api/field-referencing-rule-interface/) for extended rule contracts.
