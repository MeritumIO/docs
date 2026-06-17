+++
title = "DefinitionInterface"
description = "Fluent builder for a single service definition"
weight = 2
+++

# DefinitionInterface

`Georgeff\Kernel\DI\DefinitionInterface`

A fluent builder returned by `KernelInterface::define()`. Configures how a service is stored, aliased, and tagged.

## Methods

```php
public function share(): static;
```

Makes the service a singleton. The factory is called once; subsequent resolutions return the cached instance.

```php
public function alias(string $alias): static;
```

Registers an additional ID that resolves to the same service. Can be called multiple times.

```php
public function tag(string $tag): static;
```

Adds a tag to this service. Can be called multiple times. Tagged services can be retrieved together via [`TagRegistryInterface::getTagged()`](/kernel/api/tag-registry-interface/).

## Accessor methods

```php
public static function for(string $id, callable $factory): static;
```

Static factory. Normally not called directly — use `KernelInterface::define()` instead.

```php
public function getId(): string;
public function getFactory(): callable;
public function isShared(): bool;
public function getAliases(): string[];
public function getTags(): string[];
```

## Example

```php
$kernel->define(CacheInterface::class, fn($c) => new RedisCache(
    $c->get(RedisConnection::class)
))
->share()
->alias('cache')
->tag('resettable');
```
