+++
title = "TranslationHandler"
description = "Matches and converts a specific Throwable type into a DomainException"
weight = 3
+++

# TranslationHandler

`Meritum\StructuredLogging\TranslationHandler`

Handles a specific category of `Throwable` in the exception translation pipeline. Register implementations tagged with `'exception.translator.handlers'`.

## Methods

```php
public function matches(Throwable $exception): bool;
```

Returns `true` if this handler should process the given exception. The translator calls `matches()` on each handler in priority order and uses the first that returns `true`.

```php
public function handle(Throwable $exception): DomainException;
```

Converts the exception into a [`DomainException`](/docs/packages/structured-logging/api/domain-exception/). Only called when `matches()` returned `true`.

```php
public function priority(): int;
```

Determines evaluation order. Handlers are sorted descending — higher values run first. Use this to ensure specific handlers take precedence over general ones.

## Example

```php
use Throwable;
use Meritum\StructuredLogging\Severity;
use Meritum\StructuredLogging\TranslationHandler;
use Meritum\StructuredLogging\Exception\DomainException;

final class PdoExceptionHandler implements TranslationHandler
{
    public function matches(Throwable $exception): bool
    {
        return $exception instanceof \PDOException;
    }

    public function handle(Throwable $exception): DomainException
    {
        return new DatabaseException(
            message:  $exception->getMessage(),
            severity: Severity::Critical,
            previous: $exception
        );
    }

    public function priority(): int
    {
        return 100;
    }
}
```

```php
$kernel->define(PdoExceptionHandler::class, fn() => new PdoExceptionHandler())
       ->tag('exception.translator.handlers');
```

If no handler matches, the translator wraps the exception in `UnknownException` with error code `UNKNOWN_0000`.
