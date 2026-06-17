+++
title = "BootableModuleInterface"
description = "Module contract for running code after the container is initialized"
weight = 5
+++

# BootableModuleInterface

`Georgeff\Kernel\Module\BootableModuleInterface`

Extends [`ModuleInterface`](/kernel/api/module-interface/). Adds a `boot()` method that is called after the container is fully initialized, during the `moduleBoot` phase.

## Methods

```php
public function boot(ContainerInterface $container): void;
```

Called during the `moduleBoot` boot phase, after `containerInit` completes. The container is fully initialized at this point and all services can be resolved.

Use this when setup work requires resolving services — for example, registering listeners, warming caches, or starting background processes.

## Example

```php
use Psr\Container\ContainerInterface;
use Georgeff\Kernel\KernelInterface;
use Georgeff\Kernel\Module\BootableModuleInterface;

final class EventModule implements BootableModuleInterface
{
    public function register(KernelInterface $kernel): void
    {
        $kernel->define(EventDispatcher::class, fn() => new EventDispatcher())
            ->share();

        $kernel->define(UserCreatedListener::class, fn($c) => new UserCreatedListener(
            $c->get(MailerInterface::class)
        ));
    }

    public function boot(ContainerInterface $container): void
    {
        $dispatcher = $container->get(EventDispatcher::class);
        $dispatcher->listen(UserCreated::class, $container->get(UserCreatedListener::class));
    }
}
```
