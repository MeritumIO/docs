+++
title = "TagRegistryInterface"
description = "Resolves all services registered under a given tag"
weight = 7
+++

# TagRegistryInterface

`Georgeff\Kernel\DI\TagRegistryInterface`

Resolves all services registered under a given tag. Available in the container as `TagRegistryInterface::class` or `'kernel.tag.registry'` after boot.

## Methods

```php
public function getTagged(string $tag): array;
```

Returns an array of resolved service instances for all services tagged with `$tag`. Resolution happens on each call — if the tagged services are not shared, new instances are created.

## Example

```php
// Registration
$kernel->define(SlackNotifier::class, fn() => new SlackNotifier())->tag('notifiers');
$kernel->define(EmailNotifier::class, fn() => new EmailNotifier())->tag('notifiers');

// Resolution
$registry = $container->get(TagRegistryInterface::class);
$notifiers = $registry->getTagged('notifiers');

foreach ($notifiers as $notifier) {
    $notifier->notify($message);
}
```

A common pattern is to inject the registry into a service that fans out to all tagged implementations:

```php
final class NotificationService
{
    public function __construct(
        private readonly TagRegistryInterface $tags
    ) {}

    public function send(Notification $notification): void
    {
        foreach ($this->tags->getTagged('notifiers') as $notifier) {
            $notifier->notify($notification);
        }
    }
}
```

Register it in a module:

```php
$kernel->define(NotificationService::class, fn($c) => new NotificationService(
    $c->get(TagRegistryInterface::class)
))->share();
```
