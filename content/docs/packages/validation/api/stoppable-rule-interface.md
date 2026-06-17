+++
title = "StoppableRuleInterface"
description = "Rule contract for halting propagation to subsequent rules"
weight = 4
+++

# StoppableRuleInterface

`Meritum\Validation\StoppableRuleInterface`

Extends [`RuleInterface`](/docs/packages/validation/api/rule-interface/). Rules that implement this interface can halt evaluation of subsequent rules for the same field.

The `required`, `optional`, and `nullable` built-in rules all implement this interface — `required` stops on failure, `optional` stops when the field is absent or null, `nullable` stops when the field is null.

## Methods

```php
public function shouldPropagationStop(mixed $value, mixed ...$params): bool;
```

Called by the engine after `validate()`, regardless of whether the rule passed or failed. Returns `true` to stop evaluation of subsequent rules for this field.

Rules should be stateless — evaluate the stop condition directly from `$value` and `$params` without storing state between calls.

## Example

A rule that stops propagation on failure (so subsequent rules only run if this one passes):

```php
use Meritum\Validation\StoppableRuleInterface;

final class SlugRule implements StoppableRuleInterface
{
    public function name(): string { return 'slug'; }

    public function validate(mixed $value, mixed ...$params): bool
    {
        return is_string($value) && (bool) preg_match('/^[a-z0-9]+(?:-[a-z0-9]+)*$/', $value);
    }

    public function message(string $attribute, mixed ...$params): string
    {
        return "The {$attribute} must be a valid slug";
    }

    public function shouldPropagationStop(mixed $value, mixed ...$params): bool
    {
        return !$this->validate($value, ...$params);
    }
}
```
