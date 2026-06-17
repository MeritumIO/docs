+++
bookCollapseSection = true
weight = 3
title = "meritum/cli"
description = "Module-first console kernel for the Meritum ecosystem"
+++

# meritum/cli

`meritum/cli` extends `georgeff/kernel` with a command-based console runtime. Everything in the [kernel guide](/docs/kernels/kernel/) applies here — modules, service definitions, tagging, decoration, and the boot lifecycle all work identically.

The console runtime is named **Sage**, after Jefferson's sobriquet "the Sage of Monticello."

```
composer require meritum/cli
```

## Entry Point

`CliKernel` is the concrete implementation. Wire it up in your entry point script:

```php
use App\ModuleRepository;
use Meritum\Cli\CliKernel;
use Georgeff\Kernel\Support\Env;
use Georgeff\Kernel\Environment;

$environment = match (Env::get('APP_ENV', 'production')) {
    'local'                         => Environment::Local,
    'test', 'testing'               => Environment::Testing,
    'dev', 'develop', 'development' => Environment::Development,
    'stage', 'staging'              => Environment::Staging,
    default                         => Environment::Production,
};

$kernel = new CliKernel($environment, debug: Env::get('APP_DEBUG', false));
$kernel->addRepository(new ModuleRepository());
$kernel->boot();
exit($kernel->run());
```

`run()` delegates to `handle()`, which reads `$argv`, dispatches to the matched command, and returns an integer exit code.

## Application Name and Version

By default the application is named `Sage` and the version is auto-detected from Composer's installed package data. Override either before `boot()`:

```php
$kernel->setName('Acme Console');
$kernel->setVersion('1.0.0');
```

Both throw `KernelException` if called after `boot()`.

## Commands

Commands extend the abstract `Command` class and implement `__invoke()`. Define the command's metadata in the constructor using the fluent builder methods; implement the execution logic in `__invoke()`.

```php
use Meritum\Cli\ExitCode;
use Meritum\Cli\Command\Command;
use Meritum\Cli\Output\SageStyleInterface;

final class GreetCommand extends Command
{
    public function __construct()
    {
        $this->setName('app:greet')
             ->setDescription('Greet a user by name');

        $this->addArgument('name')
             ->description('The name to greet')
             ->optional('World');
    }

    public function __invoke(SageStyleInterface $io): ExitCode
    {
        $io->success('Hello, ' . $io->argument('name') . '!');

        return ExitCode::Success;
    }
}
```

The `__invoke()` method receives a [`SageStyleInterface`](/docs/kernels/cli/api/sage-style-interface/) instance and must return an [`ExitCode`](/docs/kernels/cli/api/exit-code/).

## Arguments

Add arguments to a command with `addArgument()`. It returns an [`Argument`](/docs/kernels/cli/api/argument/) you can configure with a fluent API:

```php
// Required argument (default)
$this->addArgument('path')
     ->description('Path to the target file');

// Optional argument with a default value
$this->addArgument('env')
     ->description('The environment to target')
     ->optional('production');

// Array argument — accepts multiple values
$this->addArgument('files')
     ->description('Files to process')
     ->array();

// With completion suggestions
$this->addArgument('driver')
     ->description('Database driver')
     ->suggest('mysql', 'pgsql', 'sqlite');
```

Read argument values inside `__invoke()` via `$io->argument()`:

```php
$path  = $io->argument('path');
$files = $io->argument('files'); // array when ->array() was called
```

## Options

Add options with `addOption()`. It returns an [`Option`](/docs/kernels/cli/api/option/) you can configure:

```php
// Flag option (no value, defaults to false)
$this->addOption('verbose')
     ->description('Enable verbose output')
     ->shortcut('v');

// Option that requires a value
$this->addOption('format')
     ->description('Output format')
     ->required()
     ->suggest('json', 'table');

// Option with an optional value
$this->addOption('output')
     ->description('Output file path')
     ->optional('/dev/stdout');

// Array option — can be passed multiple times
$this->addOption('tag')
     ->description('Tags to apply')
     ->required()
     ->array();

// Negatable flag (--feature / --no-feature)
$this->addOption('cache')
     ->description('Use the cache')
     ->negatable();
```

Read option values inside `__invoke()` via `$io->option()`:

```php
$format  = $io->option('format');
$verbose = $io->option('verbose'); // bool for flags
```

## Output

The `$io` parameter passed to `__invoke()` is a [`SageStyleInterface`](/docs/kernels/cli/api/sage-style-interface/), which extends symfony's `StyleInterface` and `OutputInterface`. All symfony output methods are available:

```php
$io->title('My Application');
$io->section('Running tasks');
$io->text('Processing...');

$io->success('Done!');
$io->error('Something went wrong');
$io->warning('This may cause issues');
$io->info('Just so you know');
$io->note('Additional context');

$io->table(
    ['Name', 'Status'],
    [['deploy', 'ok'], ['migrate', 'pending']]
);
```

Refer to the [symfony Console documentation](https://symfony.com/doc/current/console/style.html) for the full list of available output methods.

## Exit Codes

Return [`ExitCode::Success`](/docs/kernels/cli/api/exit-code/) on success and `ExitCode::Error` on failure:

```php
public function __invoke(SageStyleInterface $io): ExitCode
{
    if (!$this->canProceed()) {
        $io->error('Cannot proceed.');
        return ExitCode::Error;
    }

    $this->doWork();
    return ExitCode::Success;
}
```

`ExitCode::Success` maps to `0`; `ExitCode::Error` maps to `1`.

## Registering Commands

Commands are registered using the standard kernel tagging API. Tag any defined service with `'cli.commands'` and the CLI kernel will pick it up automatically during boot:

```php
use Georgeff\Kernel\KernelInterface;
use Georgeff\Kernel\Module\ModuleInterface;

final class AppModule implements ModuleInterface
{
    public function register(KernelInterface $kernel): void
    {
        $kernel->define(GreetCommand::class, fn() => new GreetCommand())
               ->tag('cli.commands');
    }
}
```

Constructor injection works because commands are resolved from the container:

```php
$kernel->define(SendReportCommand::class, fn($c) => new SendReportCommand(
    $c->get(MailerInterface::class),
    $c->get(ReportBuilder::class)
))->tag('cli.commands');
```

## Calling Other Commands

From within a command, call another registered command using `$this->call()`:

```php
public function __invoke(SageStyleInterface $io): ExitCode
{
    $this->call('db:migrate', $io);
    $this->call('cache:clear', $io);

    return ExitCode::Success;
}
```

Pass input parameters as the third argument:

```php
$this->call('db:seed', $io, ['--class' => 'UserSeeder']);
```

If no output is provided the called command runs silently.

## Exception Handling

By default, exceptions that escape a command bubble up and symfony renders its own error output. To handle them yourself, register an implementation of `ExceptionHandlerInterface` in the container:

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

Register it in a module:

```php
use Meritum\Cli\Exception\ExceptionHandlerInterface;

$kernel->define(ExceptionHandlerInterface::class, fn() => new ExceptionHandler());
```

The kernel checks for `ExceptionHandlerInterface::class` in the container on every `handle()` call. If present, caught exceptions are passed to it and `handle()` returns `ExitCode::Error` (1). If absent, exceptions bubble.

> `meritum/structured-logging` provides pipeline tooling that can be wired into the exception handler to capture and structure error context for observability.

## Default Container Registrations

`CliKernel` registers one service automatically during the boot sequence:

| ID | Resolves to |
|---|---|
| `'cli.app'` | The symfony `Application` instance (internal) |

Do not overwrite `'cli.app'` unless you intend to replace the entire console runtime.
