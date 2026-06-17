+++
title = "ModuleRepositoryInterface"
description = "Provides a batch of modules to the kernel"
weight = 6
+++

# ModuleRepositoryInterface

`Georgeff\Kernel\Module\ModuleRepositoryInterface`

Groups a set of modules for batch registration with the kernel via `addRepository()`. Repositories are consumed and discarded during the `moduleLoad` boot phase.

## Methods

```php
public function modules(Environment $env): array;
```

Returns `ModuleInterface[]`. The `Environment` parameter lets you conditionally include or exclude modules.

## Example

```php
use Georgeff\Kernel\Environment;
use Georgeff\Kernel\Module\ModuleRepositoryInterface;

final class ModuleRepository implements ModuleRepositoryInterface
{
    public function modules(Environment $env): array
    {
        $modules = [
            new CacheModule(),
            new DatabaseModule(),
            new AppModule(),
        ];

        if ($env === Environment::Local) {
            $modules[] = new DebugModule();
        }

        return $modules;
    }
}
```

Pass the repository to the kernel before boot:

```php
$kernel->addRepository(new ModuleRepository());
$kernel->boot();
```
