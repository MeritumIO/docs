+++
title = "KernelException"
description = "Thrown for all kernel guard violations"
weight = 9
+++

# KernelException

`Georgeff\Kernel\KernelException`

Thrown for all kernel guard violations. Extends `RuntimeException`.

## When it is thrown

| Situation | Trigger |
|---|---|
| `define()` called after boot | `KernelInterface::define()` |
| `define()` called with a reserved service ID | `KernelInterface::define()` |
| `decorate()` called after boot | `KernelInterface::decorate()` |
| `decorate()` called with a reserved service ID | `KernelInterface::decorate()` |
| `addModule()` called after boot has started | `KernelInterface::addModule()` |
| `addRepository()` called after boot has started | `KernelInterface::addRepository()` |
| The same module class is added twice | `KernelInterface::addModule()` |
| `decorate()` targets a service that has not been defined | `KernelInterface::decorate()` |

## Static helpers

`KernelException` exposes three static helpers for internal use:

```php
public static function throw(string $message): never;
public static function throwIf(bool $condition, string $message): void;
public static function throwIfNot(bool $condition, string $message): void;
```

These are part of the internal implementation and are not intended for use in application code.
