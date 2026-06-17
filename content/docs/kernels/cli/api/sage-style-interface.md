+++
title = "SageStyleInterface"
description = "Unified input/output interface passed to every command"
weight = 6
+++

# SageStyleInterface

`Meritum\Cli\Output\SageStyleInterface`

The `$io` parameter received in `Command::__invoke()`. Extends symfony's `StyleInterface` and `OutputInterface`, providing a single object for reading input and writing formatted output.

The concrete implementation is `SageStyle`, which wraps symfony's `SymfonyStyle`. All symfony style methods are available — refer to the [symfony Console style guide](https://symfony.com/doc/current/console/style.html) for the full list.

## Input methods

```php
public function argument(string $name, mixed $default = null): mixed;
```

Returns the value of the named argument. Returns `$default` if the argument was not provided and has no configured default.

```php
public function arguments(): array;
```

Returns all argument values as `array<string, mixed>`.

```php
public function option(string $name, mixed $default = null): mixed;
```

Returns the value of the named option. Returns `$default` if the option was not provided and has no configured default. Returns `bool` for flag options and negatable options.

```php
public function options(): array;
```

Returns all option values as `array<string, mixed>`.

## Common output methods (from StyleInterface)

```php
$io->title('My Application');
$io->section('Step 1');
$io->text('Line of text');

$io->success('Operation completed');
$io->error('Something failed');
$io->warning('Proceed with caution');
$io->info('Informational message');
$io->note('Side note');

$io->table(['Col A', 'Col B'], [['foo', 'bar']]);

$io->writeln('Raw output');
$io->write('Inline output');
```
