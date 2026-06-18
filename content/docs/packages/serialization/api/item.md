+++
title = "Item"
description = "Resource wrapping a single domain object"
weight = 3
+++

# Item

`Meritum\Serialization\Resource\Item`

Wraps a single domain object alongside a [`SerializerInterface`](/docs/packages/serialization/api/serializer-interface/). Implements `ItemInterface`.

## Constructor

```php
public function __construct(mixed $data, SerializerInterface $serializer)
```

`$data` is the raw domain object passed to `$serializer->serialize()` when the formatter processes the item.

## Meta

```php
public function addMeta(string $key, mixed $value): static;
public function getMeta(): array;
```

Attaches arbitrary metadata. `addMeta()` is fluent. Non-empty meta appears as a `meta` key in the envelope.

## Example

```php
use Meritum\Serialization\Resource\Item;

$resource = new Item($user, $this->serializer);
$resource->addMeta('generated_at', date('c'));

$envelope = $this->formatter->format($resource);
```
