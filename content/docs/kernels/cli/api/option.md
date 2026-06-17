+++
title = "Option"
description = "Fluent builder for a single command option"
weight = 5
+++

# Option

`Meritum\Cli\Command\Option`

Returned by `Command::addOption()`. Configures a single option (`--name`) on a command. Options are boolean flags (no value) by default.

## Value mode

```php
public function required(): self;
```

The option requires a value: `--format=json` or `--format json`.

```php
public function optional(mixed $default = null): self;
```

The option accepts a value but it is not mandatory. `$default` is used when `--option` is passed without a value.

## Modifiers

```php
public function array(): self;
```

The option can be passed multiple times: `--tag foo --tag bar`. `$io->option()` returns an array.

```php
public function negatable(): self;
```

Generates both `--option` and `--no-option` variants. `$io->option()` returns `true` or `false`.

```php
public function shortcut(string ...$shortcuts): self;
```

Registers one or more single-character shortcuts (e.g. `-v` for `--verbose`).

## Metadata

```php
public function description(string $description): self;
```

Short description shown in help output.

```php
public function suggest(string ...$suggestions): self;
```

Provides shell completion suggestions for the option's value.

## Example

```php
// Flag (no value)
$this->addOption('verbose')
     ->description('Enable verbose output')
     ->shortcut('v');

// Required value
$this->addOption('format')
     ->description('Output format')
     ->required()
     ->suggest('json', 'table', 'csv');

// Optional value with default
$this->addOption('output')
     ->description('Write output to file')
     ->optional('/dev/stdout');

// Array option
$this->addOption('tag')
     ->description('Tags to apply (can be repeated)')
     ->required()
     ->array();

// Negatable flag
$this->addOption('cache')
     ->description('Use the cache (--cache / --no-cache)')
     ->negatable();
```
