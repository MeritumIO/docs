+++
title = "KernelInterface"
description = "The primary contract for interacting with the kernel"
weight = 1
+++

# KernelInterface

`Georgeff\Kernel\KernelInterface`

The primary contract for interacting with the kernel. `Kernel` is the concrete implementation; `meritum/http` and `meritum/cli` extend it with `HttpKernelInterface` and `CliKernelInterface` respectively.

## Methods

### Boot control

```php
public function boot(): void;
```

Runs the full boot sequence. Throws if already booted.

```php
public function shutdown(): void;
```

Runs shutdown callbacks. No-op if already shutdown or not yet booted.

```php
public function isBooting(): bool;
```

Returns `true` during the boot sequence (between `preBoot` and `postBoot`).

```php
public function isBooted(): bool;
```

Returns `true` after `boot()` completes.

```php
public function isShutdown(): bool;
```

Returns `true` after `shutdown()` completes.

---

### Lifecycle hooks

```php
public function onBooting(callable $callback): static;
```

Registers a callback to run at the start of `boot()`, before any modules are loaded. The callback receives `KernelInterface`.

```php
public function onBooted(callable $callback): static;
```

Registers a callback to run after `boot()` completes. The callback receives `KernelInterface`.

```php
public function onShutdown(callable $callback): static;
```

Registers a callback to run at the start of `shutdown()`. The callback receives `KernelInterface`.

```php
public function afterShutdown(callable $callback): static;
```

Registers a callback to run after `shutdown()` completes. The callback receives `KernelInterface`.

Multiple callbacks can be registered for each hook; they run in registration order.

---

### Service definitions

```php
public function define(string $id, callable $factory): DefinitionInterface;
```

Registers a service definition and returns a [`DefinitionInterface`](/kernel/api/definition-interface/) for further configuration. The factory signature is `callable(ContainerInterface): mixed`.

Throws `KernelException` if called after `boot()` or if `$id` is a reserved service.

```php
public function tag(string $id, array $tags): static;
```

Adds tags to an already-defined service. `$tags` is a `string[]`. Silent no-op if `$id` has not been defined.

```php
public function decorate(string $id, callable $decorator): static;
```

Wraps an existing service. The decorator signature is `callable(mixed $inner, ContainerInterface $c): mixed`.

Throws `KernelException` if called after `boot()` or if `$id` is a reserved service.

---

### Module registration

```php
public function addModule(ModuleInterface $module): static;
```

Registers a single module. Throws `KernelException` if the same class has already been added, or if called after boot has started.

```php
public function addRepository(ModuleRepositoryInterface $repository): static;
```

Registers a module repository. Throws `KernelException` if called after boot has started.

---

### Container and environment

```php
public function getContainer(): \Psr\Container\ContainerInterface;
```

Returns the PSR-11 container. Only available after `boot()`.

```php
public function getEnvironment(): string;
```

Returns the current environment as a string (`'local'`, `'production'`, `'staging'`, `'development'`, `'testing'`).

```php
public function isDebug(): bool;
```

Returns `true` if debug mode is enabled.

```php
public function getStartTime(): float;
```

Returns the kernel's construction timestamp. Returns `-INF` unless debug mode is enabled.

---

## Reserved services

The following service IDs cannot be overwritten via `define()` or wrapped via `decorate()`:

| ID | Resolves to |
|---|---|
| `KernelInterface::class` | The kernel instance |
| `'kernel'` | Alias for `KernelInterface::class` |
| `DI\TagRegistryInterface::class` | The tag registry |
| `'kernel.tag.registry'` | Alias for `TagRegistryInterface::class` |
| `'kernel.debug'` | `bool` — debug mode flag |
| `'kernel.environment'` | `string` — environment value |

`'kernel.config'` is also reserved and registered during boot from module `config()` contributions.
