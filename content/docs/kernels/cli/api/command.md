+++
title = "Command"
description = "Abstract base class for all CLI commands"
weight = 3
+++

# Command

`Meritum\Cli\Command\Command`

Abstract base class that implements [`CommandInterface`](/docs/kernels/cli/api/command-interface/). Extend this class to create commands. Configure the command in the constructor using the fluent builder methods; implement `__invoke()` to define the execution logic.

## Metadata builders

```php
public function setName(string $name): static;
```

Sets the command name (e.g. `'app:greet'`). Required — the kernel throws if a command has no name.

```php
public function setDescription(string $description): static;
```

Short one-line description shown in the command list.

```php
public function setHelp(string $help): static;
```

Extended help text shown when `--help` is passed.

```php
public function addAlias(string $alias): static;
```

Registers an alternative name the command can be called by.

```php
public function shouldHide(bool $hide = true): static;
```

Hides the command from the command list. Hidden commands are still executable by name.

All metadata builder methods throw `LogicException` if called after the kernel has been set (i.e., after the command is registered and the boot sequence runs).

## Input schema builders

```php
public function addArgument(string $name): Argument;
```

Adds an argument to the command and returns an [`Argument`](/docs/kernels/cli/api/argument/) for further configuration. Arguments are required by default.

```php
public function addOption(string $name): Option;
```

Adds an option to the command and returns an [`Option`](/docs/kernels/cli/api/option/) for further configuration. Options are flags (no value) by default.

Both throw `LogicException` if called after the kernel has been set.

## Calling other commands

```php
public function call(
    string $command,
    ?SageStyleInterface $output = null,
    array $parameters = []
): int;
```

Dispatches another registered command by name. `$output` is passed to the called command; defaults to silent output if omitted. `$parameters` are merged into the input as if they were passed on the command line. Returns the integer exit code of the called command.

Throws `LogicException` if called before the kernel has been set.

## Example

```php
use Meritum\Cli\ExitCode;
use Meritum\Cli\Command\Command;
use Meritum\Cli\Output\SageStyleInterface;

final class PublishCommand extends Command
{
    public function __construct(private readonly Publisher $publisher)
    {
        $this->setName('app:publish')
             ->setDescription('Publish all pending drafts');

        $this->addArgument('target')
             ->description('Target environment')
             ->optional('production')
             ->suggest('production', 'staging');

        $this->addOption('dry-run')
             ->description('Preview changes without publishing')
             ->shortcut('n');
    }

    public function __invoke(SageStyleInterface $io): ExitCode
    {
        $target = $io->argument('target');
        $dry    = $io->option('dry-run');

        if ($dry) {
            $io->note("Dry run — no changes will be made to {$target}.");
        }

        $this->publisher->publish($target, dry: $dry);
        $io->success('Done.');

        return ExitCode::Success;
    }
}
```
