+++
title = "Built-in Rules"
description = "Complete reference for all rules shipped with meritum/validation"
weight = 6
+++

# Built-in Rules

All rules shipped with `meritum/validation`. Rules listed as stoppable halt propagation to subsequent rules for the same field when the stop condition is met.

## Presence / flow

| Rule | Params | Stop condition | Description |
|---|---|---|---|
| `required` | — | on failure | Fails if absent, `null`, or empty string |
| `optional` | — | if absent or `null` | Always passes; skips subsequent rules if absent or `null` |
| `nullable` | — | if `null` | Always passes; skips subsequent rules if `null` |

## Type

| Rule | Params | Description |
|---|---|---|
| `string` | — | `is_string()` |
| `integer` | — | `is_int()` |
| `float` | — | `is_float()` — integers do not pass; use `numeric` for numeric strings |
| `boolean` | — | `is_bool()` |
| `array` | — | `is_array()` |
| `numeric` | — | `is_numeric()` — accepts integers, floats, and numeric strings |

## String

| Rule | Params | Description |
|---|---|---|
| `alpha` | — | Alphabetic characters only |
| `alphaNum` | — | Alphanumeric characters only |
| `email` | — | Valid email address |
| `url` | — | Valid URL |
| `uuid` | — | Valid UUID |
| `ip` | — | Valid IP address (v4 or v6) |
| `ipv4` | — | Valid IPv4 address |
| `ipv6` | — | Valid IPv6 address |
| `regex` | `pattern` | Matches a regular expression pattern |
| `lengthMin` | `min` | String length ≥ min (`mb_strlen`) |
| `lengthMax` | `max` | String length ≤ max (`mb_strlen`) |
| `lengthBetween` | `min, max` | String length between min and max inclusive (`mb_strlen`) |

## Numeric

| Rule | Params | Description |
|---|---|---|
| `min` | `min` | Value ≥ min (numeric) |
| `max` | `max` | Value ≤ max (numeric) |
| `between` | `min, max` | Value between min and max inclusive (numeric) |

## Comparison

| Rule | Params | Description |
|---|---|---|
| `equals` | `value` | Strict equality to `value` |
| `notEquals` | `value` | Strict inequality to `value` |
| `in` | `val, ...` | Value is in the list (strict comparison) |
| `notIn` | `val, ...` | Value is not in the list (strict comparison) |
| `sameAs` | `field` | Strict equality to another field's value |
| `differentFrom` | `field` | Strict inequality from another field's value |

`sameAs` and `differentFrom` are field-referencing rules — see [`FieldReferencingRuleInterface`](/docs/packages/validation/api/field-referencing-rule-interface/).

## Date

| Rule | Params | Description |
|---|---|---|
| `date` | — | Parseable date string (validated with `checkdate`) |
| `dateFormat` | `format` | Date string matching a PHP `date()` format (e.g. `Y-m-d`) |

## Schema examples

```php
// Presence
'name'  => ['required', 'string'],
'bio'   => ['nullable', 'string'],
'alias' => ['optional', 'string'],

// Type + length
'username' => ['required', 'string', 'lengthBetween' => [3, 32]],
'age'      => ['required', 'integer', 'between' => [13, 120]],
'price'    => ['required', 'numeric', 'min' => 0],

// Enum-like
'status' => ['required', 'in' => ['draft', 'published', 'archived']],

// Format
'email'    => ['required', 'email'],
'website'  => ['optional', 'url'],
'slug'     => ['required', 'regex' => '/^[a-z0-9-]+$/'],
'birthday' => ['optional', 'dateFormat' => 'Y-m-d'],

// Cross-field
'password_confirmation' => ['required', 'sameAs' => 'password'],
'new_password'          => ['required', 'differentFrom' => 'current_password'],
```
