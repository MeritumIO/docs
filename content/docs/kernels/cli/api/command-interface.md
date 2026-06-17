+++
title = "CommandInterface"
description = "Contract for all CLI commands"
weight = 2
+++

# CommandInterface

`Meritum\Cli\Command\CommandInterface`

The contract every command must satisfy. In practice, extend the abstract [`Command`](/docs/kernels/cli/api/command/) base class rather than implementing this interface directly.

## Execution

```php
public function __invoke(SageStyleInterface $io): ExitCode;
```

Entry point called when the command is dispatched. Receives a [`SageStyleInterface`](/docs/kernels/cli/api/sage-style-interface/) for reading input and writing output. Must return an [`ExitCode`](/docs/kernels/cli/api/exit-code/).

## Metadata accessors

```php
public function getName(): string;
public function getDescription(): string;
public function getHelp(): string;
public function getAliases(): string[];
public function isHidden(): bool;
```

Called by the internal symfony adapter to configure the command before it is registered with the console application.

## Input schema accessors

```php
/** @return Argument[] */
public function getArguments(): array;

/** @return Option[] */
public function getOptions(): array;
```

Return the argument and option definitions that describe the command's input schema.

## Lifecycle

```php
public function setKernel(CliKernelInterface $kernel): static;
```

Called by the CLI kernel after the command is resolved from the container. Provides access to the kernel instance, which is required by `Command::call()`. Must not be called by user code.
