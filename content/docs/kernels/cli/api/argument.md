+++
title = "Argument"
description = "Fluent builder for a single command argument"
weight = 4
+++

# Argument

`Meritum\Cli\Command\Argument`

Returned by `Command::addArgument()`. Configures a single positional argument on a command. Arguments are required by default.

## Methods

```php
public function optional(mixed $default = null): self;
```

Makes the argument optional. If the user does not supply a value, `$default` is used.

```php
public function array(): self;
```

Marks the argument as an array. The user can provide multiple values; `$io->argument()` returns an array.

```php
public function description(string $description): self;
```

Short description shown in help output.

```php
public function suggest(string ...$suggestions): self;
```

Provides shell completion suggestions for this argument.

## Example

```php
// Required
$this->addArgument('path')
     ->description('Path to the target file');

// Optional with default
$this->addArgument('env')
     ->description('Target environment')
     ->optional('production')
     ->suggest('production', 'staging', 'local');

// Array argument
$this->addArgument('files')
     ->description('Files to process')
     ->array();
```
