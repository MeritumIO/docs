+++
title = "Collection"
description = "Resource wrapping an iterable of domain objects"
weight = 4
+++

# Collection

`Meritum\Serialization\Resource\Collection`

Wraps an iterable of domain objects alongside a [`SerializerInterface`](/docs/packages/serialization/api/serializer-interface/). Implements `CollectionInterface`. Supports cursor-based and offset-based pagination.

## Constructor

```php
/** @param iterable<mixed> $data */
public function __construct(iterable $data, SerializerInterface $serializer)
```

The serializer is called once per element in `$data` when the formatter builds the collection.

## Pagination

```php
public function setCursor(CursorInterface $cursor): static;
```

Attaches cursor pagination. Clears any previously set paginator. Produces `limit`, `previous`, and `next` keys in the envelope.

```php
public function setPaginator(PaginatorInterface $paginator): static;
```

Attaches offset pagination. Clears any previously set cursor. Produces `total`, `count`, `limit`, `current`, and `last` keys in the envelope.

See [`CursorInterface`](/docs/packages/serialization/api/cursor-interface/) and [`PaginatorInterface`](/docs/packages/serialization/api/paginator-interface/).

## Meta

```php
public function addMeta(string $key, mixed $value): static;
public function getMeta(): array;
```

Attaches arbitrary metadata. Non-empty meta appears as a `meta` key in the envelope alongside `data`.

## Example

```php
use Meritum\Serialization\Resource\Collection;

$resource = new Collection($this->users->paginate($page, $perPage), $this->serializer);
$resource->setPaginator($paginator);
$resource->addMeta('cached', false);

$envelope = $this->formatter->format($resource);

return new JsonResponse($envelope);
```
