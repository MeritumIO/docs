+++
title = "CliKernelInterface"
description = "The CLI kernel contract — extends KernelInterface with console-specific methods"
weight = 1
+++

# CliKernelInterface

`Meritum\Cli\CliKernelInterface`

Extends `KernelInterface` (via `RunnableKernelInterface`) with methods for naming the application and dispatching console commands. All kernel methods — `define()`, `decorate()`, `addModule()`, `addRepository()`, `boot()`, `getContainer()`, etc. — are available through the inherited interface.

## Application metadata

```php
public function setName(string $name): static;
public function getName(): string;
```

Sets and retrieves the application name shown in the console header. Defaults to `'Sage'`. `setName()` throws `KernelException` if called after `boot()`.

```php
public function setVersion(string $version): static;
public function getVersion(): string;
```

Sets and retrieves the version string shown in the console header. Defaults to the installed Composer version of `meritum/cli`. `setVersion()` throws `KernelException` if called after `boot()`.

## Dispatch

```php
public function handle(
    ?InputInterface $input = null,
    ?OutputInterface $output = null
): int;
```

Dispatches the input to the matched command. `$input` defaults to `ArgvInput` (reads `$argv`); `$output` defaults to `ConsoleOutput`. Returns an integer exit code (`0` for success, `1` for error).

If an exception escapes the command and an `ExceptionHandlerInterface` is registered in the container, the handler is called and `1` is returned. If no handler is registered, the exception is re-thrown.

Throws `KernelException` if the kernel has not been booted or has already been shut down.

```php
public function run(): int;
```

Delegates to `handle()` with default input and output. The standard entry point — call `exit($kernel->run())` in your entry script.
