+++
bookCollapseSection = true
weight = 1
title = "georgeff/kernel"
description = "Dependency injection, modules, and the boot lifecycle"
+++

# georgeff/kernel

The kernel is the foundation of every Meritum application. It manages dependency injection, wires modules together, and controls the boot lifecycle. Every Meritum package — including `meritum/http` and `meritum/cli` — extends the kernel rather than replacing it, so the concepts here apply everywhere.

The design is built around a strict separation between two phases: a **boot phase** where services are registered and the dependency graph is assembled, and a **run phase** where that graph is sealed and used. For the architectural reasoning behind this, see [Two Phases, Two APIs](https://dev.to/georgeff/two-phases-two-apis-425j).

```
composer require georgeff/kernel
```

## Defining Services

Services are registered with `define()`, which returns a [`DefinitionInterface`](/kernel/api/definition-interface/) you can configure with a fluent API.

```php
$kernel->define(CacheInterface::class, fn() => new RedisCache());
```

By default, the factory is called fresh on every resolution. Use `share()` to make the service a singleton:

```php
$kernel->define(CacheInterface::class, fn() => new RedisCache())
    ->share();
```

Register an alias so the service can be resolved by multiple IDs:

```php
$kernel->define(CacheInterface::class, fn() => new RedisCache())
    ->share()
    ->alias('cache');
```

The factory receives a `Psr\Container\ContainerInterface` instance as its first argument, which lets you pull in other services:

```php
$kernel->define(UserRepository::class, fn($c) => new UserRepository(
    $c->get(DatabaseInterface::class)
));
```

> `define()` throws `KernelException` if called after `boot()`. The service graph is sealed at boot time.

## Service Tagging

Tags group related services so they can be retrieved together. Tag a service at definition time:

```php
$kernel->define(SlackNotifier::class, fn() => new SlackNotifier($config))
    ->tag('notifiers');

$kernel->define(EmailNotifier::class, fn() => new EmailNotifier($config))
    ->tag('notifiers');
```

Or tag after defining using `tag()` on the kernel directly:

```php
$kernel->define(SlackNotifier::class, fn() => new SlackNotifier($config));
$kernel->tag(SlackNotifier::class, ['notifiers']);
```

> Calling `tag()` for an ID that has not been defined is a silent no-op. Always define the service first.

Inside the container (available after boot), retrieve all tagged services via the [`TagRegistryInterface`](/kernel/api/tag-registry-interface/):

```php
$registry = $container->get(TagRegistryInterface::class);
$notifiers = $registry->getTagged('notifiers'); // array of resolved instances
```

The tag registry is available as `TagRegistryInterface::class` or `'kernel.tag.registry'`.

## Building Modules

Modules are where you register services, provide configuration, and hook into the boot sequence. A module implements one or more of the three module interfaces.

### ModuleInterface

The base contract. Every module must implement `register()`:

```php
use Georgeff\Kernel\KernelInterface;
use Georgeff\Kernel\Module\ModuleInterface;

final class CacheModule implements ModuleInterface
{
    public function register(KernelInterface $kernel): void
    {
        $kernel->define(CacheInterface::class, fn() => new RedisCache())
            ->share();
    }
}
```

### ConfigurableModuleInterface

Extends `ModuleInterface` with a `config()` method that contributes entries to the kernel's shared config array (`kernel.config`). The `Environment` is passed so you can return environment-specific values:

```php
use Georgeff\Kernel\Environment;
use Georgeff\Kernel\Support\Env;
use Georgeff\Kernel\Module\ConfigurableModuleInterface;

final class AppModule implements ConfigurableModuleInterface
{
    public function register(KernelInterface $kernel): void
    {
        $kernel->define(Mailer::class, fn($c) => new Mailer(
            $c->get('kernel.config')['mail']
        ))->share();
    }

    public function config(Environment $env): array
    {
        return [
            'mail' => [
                'host'     => Env::get('MAIL_HOST', 'localhost'),
                'port'     => (int) Env::get('MAIL_PORT', 25),
                'from'     => Env::get('MAIL_FROM', 'app@example.com'),
            ],
        ];
    }
}
```

Config from all modules is merged and registered as `kernel.config` in the container. It is available after boot.

### BootableModuleInterface

Extends `ModuleInterface` with a `boot()` method called after the container is fully initialized. Use this when you need to resolve services from the container to perform setup work:

```php
use Psr\Container\ContainerInterface;
use Georgeff\Kernel\Module\BootableModuleInterface;

final class SchedulerModule implements BootableModuleInterface
{
    public function register(KernelInterface $kernel): void
    {
        $kernel->define(Scheduler::class, fn() => new Scheduler())->share();
    }

    public function boot(ContainerInterface $container): void
    {
        $container->get(Scheduler::class)->start();
    }
}
```

A single module can implement any combination of the three interfaces.

## Providing Modules

Pass modules to the kernel individually or batch them through a [`ModuleRepositoryInterface`](/kernel/api/module-repository-interface/):

```php
// Individually
$kernel->addModule(new CacheModule());
$kernel->addModule(new AppModule());

// Via a repository
$kernel->addRepository(new ModuleRepository());
```

`ModuleRepository` returns an array of module instances from `modules(Environment $env)`, which lets you conditionally include modules by environment:

```php
final class ModuleRepository implements ModuleRepositoryInterface
{
    public function modules(Environment $env): array
    {
        $modules = [
            new CacheModule(),
            new AppModule(),
        ];

        if ($env === Environment::Local) {
            $modules[] = new DebugModule();
        }

        return $modules;
    }
}
```

Adding the same module class twice — whether directly or through a repository — throws `KernelException`.

## Boot Lifecycle

Call `boot()` to start the kernel. The boot sequence runs through eight phases in order:

| Phase | What happens |
|---|---|
| `preBoot` | `onBooting()` callbacks run |
| `moduleLoad` | Repositories are flattened; `config()` results are merged into `kernel.config`; `$booting` becomes `true` |
| `moduleRegistration` | `register()` is called on each module |
| `serviceDecoration` | Decorators are applied |
| `serviceRegistration` | Definitions are pushed into the container; tags are indexed |
| `containerInit` | The container is finalized and made available via `getContainer()` |
| `moduleBoot` | `boot()` is called on each `BootableModuleInterface` module |
| `postBoot` | `onBooted()` callbacks run |

### Lifecycle Hooks

Register callbacks to run at specific points in the lifecycle:

```php
$kernel->onBooting(function (KernelInterface $kernel): void {
    // Runs before any modules are loaded
});

$kernel->onBooted(function (KernelInterface $kernel): void {
    // Runs after the container is ready and all modules have booted
    $container = $kernel->getContainer();
});

$kernel->onShutdown(function (KernelInterface $kernel): void {
    // Runs when shutdown() is called
});

$kernel->afterShutdown(function (KernelInterface $kernel): void {
    // Runs after shutdown completes
});
```

Multiple callbacks can be registered for each hook; they run in registration order.

## Service Decoration

Decoration wraps an existing service without replacing its definition. The decorator receives the original service and the container:

```php
$kernel->define(LoggerInterface::class, fn() => new FileLogger('/var/log/app.log'))
    ->share();

$kernel->decorate(LoggerInterface::class, function (LoggerInterface $inner, $container): LoggerInterface {
    return new CorrelationIdLogger($inner, $container->get(CorrelationId::class));
});
```

Decoration is two-phase: decorators are registered eagerly but applied lazily at the `serviceDecoration` boot phase. Nothing runs until `boot()`.

> `decorate()` throws `KernelException` if called after `boot()` or if the target ID is a reserved service.

## The Immutable Service Graph

Once `boot()` completes, the service graph is sealed. Calling `define()` or `decorate()` after boot throws `KernelException`. Similarly, `addModule()` and `addRepository()` throw if called after boot has started.

This guarantee means that the container you receive from `getContainer()` reflects the complete, final set of services. Nothing can be added or changed at runtime.

## Debug Mode

Enable debug mode by passing `debug: true` to the kernel constructor:

```php
$kernel = new Kernel(Environment::Local, debug: true);
```

In debug mode, the kernel records boot phase durations and tracks every container resolution — when a service was resolved, how many times, and how long each resolution took. Services that implement [`DebuggableInterface`](/kernel/api/debuggable-interface/) automatically contribute their own debug output to the aggregated report.

Dump the full debug snapshot at any point after boot:

```php
$info = $kernel->getDebugInfo();
```

The returned array includes:

- `bootProfile` — per-phase timing from the boot sequence
- `container` — resolution counts, cumulative times, and unresolved services
- Debug output from any sub-system that implements `DebuggableInterface`

The `DebuggableInterface` auto-aggregation means that as the ecosystem grows, new debuggable sub-systems appear in the report without any changes to the kernel itself. Implement `getDebugInfo()` on any service registered in the container and it will be included automatically.

> `getStartTime()` returns the kernel's construction timestamp only in debug mode. Outside debug mode it returns `-INF`.
