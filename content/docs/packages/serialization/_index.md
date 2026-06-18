+++
bookCollapseSection = true
weight = 4
title = "meritum/serialization"
description = "Pluggable object serialization with item, collection, and pagination support"
+++

# meritum/serialization

`meritum/serialization` transforms domain objects into structured array output ready for JSON responses. The pipeline is: write a serializer for each model type, wrap data in an `Item` or `Collection` resource, then pass it to the `FormatterInterface` to get an `EnvelopeInterface` back.

```
composer require meritum/serialization
```

## Module Registration

Add `SerializationModule` to your module repository:

```php
use Meritum\Serialization\SerializationModule;

final class ModuleRepository implements ModuleRepositoryInterface
{
    public function modules(Environment $env): array
    {
        return [
            new SerializationModule(),
            // ...
        ];
    }
}
```

The module registers `FormatterInterface::class` in the container using `DataArrayStrategy` by default. Inject `FormatterInterface` into your handlers and actions.

## Writing a Serializer

Implement `SerializerInterface` for each model type you need to serialize:

```php
use Meritum\Serialization\SerializerInterface;

final class UserSerializer implements SerializerInterface
{
    public function serialize(mixed $data): array
    {
        return [
            'id'    => $data->id,
            'name'  => $data->name,
            'email' => $data->email,
        ];
    }
}
```

The serializer receives whatever was passed as the data — typically a model or value object. It returns a flat or nested `array<string, mixed>`.

## Single Item

Wrap a single object in `Item` and pass it to the formatter:

```php
use Meritum\Serialization\Resource\Item;
use Meritum\Serialization\FormatterInterface;

final class ShowUserHandler implements RequestHandlerInterface
{
    public function __construct(
        private readonly UserRepository $users,
        private readonly UserSerializer $serializer,
        private readonly FormatterInterface $formatter
    ) {}

    public function handle(ServerRequestInterface $request): ResponseInterface
    {
        $user = $this->users->find($request->getAttribute('id'));

        $resource = new Item($user, $this->serializer);
        $envelope = $this->formatter->format($resource);

        return new JsonResponse($envelope);
    }
}
```

With the default `DataArrayStrategy`, the response body is:

```json
{
  "data": {
    "id": 1,
    "name": "Jane Smith",
    "email": "jane@example.com"
  }
}
```

## Collection

Wrap an iterable in `Collection`:

```php
use Meritum\Serialization\Resource\Collection;

$resource = new Collection($this->users->all(), $this->serializer);
$envelope = $this->formatter->format($resource);

return new JsonResponse($envelope);
```

Both strategies produce the same collection shape:

```json
{
  "data": [
    { "id": 1, "name": "Jane Smith", "email": "jane@example.com" },
    { "id": 2, "name": "John Doe",   "email": "john@example.com" }
  ]
}
```

## Pagination

Attach pagination data to a `Collection` before formatting. The two strategies are cursor-based and offset-based — they are mutually exclusive on a collection.

### Cursor pagination

Implement `CursorInterface` and call `setCursor()`:

```php
$resource = new Collection($users, $this->serializer);
$resource->setCursor($cursor);

$envelope = $this->formatter->format($resource);
```

Output:

```json
{
  "limit": 25,
  "previous": "eyJpZCI6MTB9",
  "next": "eyJpZCI6MzV9",
  "data": [...]
}
```

`previous` and `next` are `null` when there is no adjacent page.

### Offset pagination

Implement `PaginatorInterface` and call `setPaginator()`:

```php
$resource = new Collection($users, $this->serializer);
$resource->setPaginator($paginator);

$envelope = $this->formatter->format($resource);
```

Output:

```json
{
  "total": 143,
  "count": 25,
  "limit": 25,
  "current": 2,
  "last": 6,
  "data": [...]
}
```

`setCursor()` and `setPaginator()` are mutually exclusive — calling one clears the other.

## Meta

Both `Item` and `Collection` accept arbitrary metadata via `addMeta()`:

```php
$resource = new Item($user, $this->serializer);
$resource->addMeta('generated_at', date('c'));
$resource->addMeta('version', '2');
```

Meta appears as a `meta` key in the envelope alongside `data`:

```json
{
  "data": { ... },
  "meta": {
    "generated_at": "2026-06-17T10:00:00+00:00",
    "version": "2"
  }
}
```

`addMeta()` is fluent and can be chained.

## Output Strategies

The strategy controls the envelope shape for items. Both strategies produce identical output for collections.

### DataArrayStrategy (default)

Item output wraps serialized fields in a `data` key:

```json
{
  "data": {
    "id": 1,
    "name": "Jane Smith"
  }
}
```

### ArrayStrategy

Item output places serialized fields at the root:

```json
{
  "id": 1,
  "name": "Jane Smith"
}
```

To use `ArrayStrategy`, register it as `StrategyInterface::class` in a module before the formatter is resolved:

```php
use Meritum\Serialization\Strategy\ArrayStrategy;
use Meritum\Serialization\Strategy\StrategyInterface;

$kernel->define(StrategyInterface::class, fn() => new ArrayStrategy());
```

`SerializationModule` checks the container for `StrategyInterface::class` at resolution time and uses it if present; otherwise it falls back to `DataArrayStrategy`.

### Custom strategy

Implement `StrategyInterface` to produce any envelope shape you need:

```php
use Meritum\Serialization\Envelope;
use Meritum\Serialization\EnvelopeInterface;
use Meritum\Serialization\Resource\ItemInterface;
use Meritum\Serialization\Resource\CollectionInterface;
use Meritum\Serialization\Strategy\StrategyInterface;

final class JsonApiStrategy implements StrategyInterface
{
    public function item(ItemInterface $item): EnvelopeInterface
    {
        return new Envelope([
            'data' => [
                'type'       => 'user',
                'attributes' => $item->getSerializer()->serialize($item->getData()),
            ],
        ]);
    }

    public function collection(CollectionInterface $collection): EnvelopeInterface
    {
        $items = [];

        foreach ($collection->getData() as $record) {
            $items[] = [
                'type'       => 'user',
                'attributes' => $collection->getSerializer()->serialize($record),
            ];
        }

        return new Envelope(['data' => $items]);
    }
}
```

Register it as `StrategyInterface::class` and it will be picked up by the formatter automatically.
