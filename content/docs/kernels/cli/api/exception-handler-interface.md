+++
title = "ExceptionHandlerInterface"
description = "Converts unhandled exceptions into console output"
weight = 7
+++

# ExceptionHandlerInterface

`Meritum\Cli\Exception\ExceptionHandlerInterface`

Handles exceptions that escape a command's `__invoke()` method. Register an implementation as `ExceptionHandlerInterface::class` in the container to activate it. If no implementation is registered, exceptions are re-thrown.

Unlike the HTTP kernel's exception handler, this interface returns `void` — the CLI kernel always returns `ExitCode::Error` (1) when a handled exception occurs.

## Methods

```php
public function handle(Throwable $exception, SageStyleInterface $io): void;
```

Called by the kernel when an exception escapes the command. Write error output via `$io`. Do not throw from this method.

## Example

```php
use Throwable;
use Meritum\Cli\Output\SageStyleInterface;
use Meritum\Cli\Exception\ExceptionHandlerInterface;

final class ExceptionHandler implements ExceptionHandlerInterface
{
    public function handle(Throwable $exception, SageStyleInterface $io): void
    {
        $io->error($exception->getMessage());
    }
}
```

Register in a module:

```php
use Meritum\Cli\Exception\ExceptionHandlerInterface;

$kernel->define(ExceptionHandlerInterface::class, fn() => new ExceptionHandler());
```
