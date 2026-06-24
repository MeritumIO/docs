+++
bookCollapseSection = true
weight = 1
title = "meritum/validation"
description = "Array-based input validation for the Meritum ecosystem"
+++

# meritum/validation

`meritum/validation` is a schema-driven validator that integrates with the kernel's module system. Rules are registered as tagged services, so the built-in set can be extended or overridden without touching the engine.

```
composer require meritum/validation
```

## Module Registration

Add `ValidationModule` to your module repository:

```php
use Meritum\Validation\ValidationModule;

final class ModuleRepository implements ModuleRepositoryInterface
{
    public function modules(Environment $env): array
    {
        return [
            new ValidationModule(),
            // ...
        ];
    }
}
```

The module registers the `Validator` service and all built-in rules. Inject `Validator` wherever validation is needed:

```php
use Meritum\Validation\Validator;

final class CreateUserHandler implements RequestHandlerInterface
{
    public function __construct(private readonly Validator $validator) {}

    public function handle(ServerRequestInterface $request): ResponseInterface
    {
        $result = $this->validator->validate([
            'name'  => ['required', 'string', 'lengthMax' => 100],
            'email' => ['required', 'email'],
            'age'   => ['required', 'integer', 'min' => 18],
        ], $request->getParsedBody());

        if (!$result->passed()) {
            return new JsonResponse(['errors' => $result->getErrors()], 422);
        }

        // ...
    }
}
```

## Schema Syntax

The first argument to `validate()` is a schema: an `array<string, array>` mapping field names to a list of rules.

Rules without parameters are listed as values with integer keys:

```php
'field' => ['required', 'string']
```

Rules that take parameters use the rule name as the key:

```php
'age'   => ['integer', 'min' => 18, 'max' => 120],
'price' => ['numeric', 'between' => [0.01, 9999.99]],
'name'  => ['string', 'lengthBetween' => [2, 100]],
```

For rules that accept a list of allowed values, pass an array:

```php
'status' => ['required', 'in' => ['active', 'inactive', 'pending']],
```

Rules are evaluated in order. Errors are keyed by field name; a field can accumulate multiple error messages if multiple rules fail.

## Presence and Nullability

Fields without `required` are implicitly optional — type and format rules pass silently when a field is absent. **Rule order matters — `nullable` must come before the rules it gates.**

| Rules | Absent from input | `null` | Present |
|---|---|---|---|
| `['string']` | passes — skipped | fails — string | validates |
| `['required', 'string']` | fails — required | fails — required | validates |
| `['nullable', 'string']` | passes — skipped | passes — skipped | validates |
| `['nullable', 'required', 'string']` | fails — required | passes — skipped | validates |

`required` fails if the field is absent, null, or an empty string, then stops propagation so only one error is reported.

`nullable` always passes, but stops propagation if the field is null. Subsequent rules are only evaluated when the field is non-null. Use `nullable` + `required` when a field must be present but null is an acceptable value.

## Nested Attributes

Use dot notation to validate nested array fields:

```php
$result = $validator->validate([
    'address.street' => ['required', 'string'],
    'address.city'   => ['required', 'string'],
    'address.zip'    => ['required', 'string', 'lengthMin' => 5],
], $input);
```

The engine walks the input array by splitting on `.`, so `'address.city'` resolves `$input['address']['city']`. Missing intermediate keys produce a `Missing` sentinel, which `required` fails on.

## Wildcard Attributes

Use `*` to validate every element of an array field:

```php
$result = $validator->validate([
    'tags'        => ['required', 'array'],
    'tags.*.name' => ['required', 'string', 'lengthMax' => 50],
    'tags.*.slug' => ['string'],
], $input);
```

The engine expands `tags.*` into concrete paths for each key present in `$input['tags']` — `tags.0.name`, `tags.1.name`, and so on. Errors are keyed by the concrete path.

If the parent (`tags`) is absent or not an array, the wildcard expansion is silently skipped — no errors are generated for the child paths.

Nested wildcards are supported:

```php
'order.*.items.*.sku' => ['required', 'string'],
```

## Field-Referencing Rules

`sameAs` and `differentFrom` compare the validated field against another field in the input:

```php
$result = $validator->validate([
    'password'              => ['required', 'string', 'lengthMin' => 8],
    'password_confirmation' => ['required', 'sameAs' => 'password'],
], $input);
```

The rule receives the target field's name as its parameter. The engine resolves the target value from the input at validation time and performs a strict comparison.

## Custom Rules

Implement `RuleInterface` and tag the service with `'validation.rules'`:

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

Register it in a module:

```php
$kernel->define(SlugRule::class, fn() => new SlugRule())
       ->tag('validation.rules');
```

The engine picks up the new rule automatically. Use it in any schema:

```php
'slug' => ['required', 'slug'],
```

To override a built-in rule, register a new class that returns the same `name()`. The last-registered rule with a given name wins.

### Stopping propagation

If your rule should halt evaluation of subsequent rules when a condition is met, implement `StoppableRuleInterface`:

```php
use Meritum\Validation\StoppableRuleInterface;

final class SlugRule implements StoppableRuleInterface
{
    // name(), validate(), message() ...

    public function shouldPropagationStop(mixed $value, mixed ...$params): bool
    {
        return !$this->validate($value, ...$params);
    }
}
```

`shouldPropagationStop()` is called after `validate()` regardless of pass or fail.

### Field-referencing rules

If your rule needs to compare the validated value against another field in the input, implement `FieldReferencingRuleInterface`. The schema parameter is the target field name; the engine resolves its value before calling `validate()`:

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

Use it in a schema:

```php
'end_date' => ['required', 'date', 'greaterThanField' => 'start_date'],
```
