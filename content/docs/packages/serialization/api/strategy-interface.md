+++
title = "StrategyInterface"
description = "Controls the envelope shape produced by the formatter"
weight = 8
+++

# StrategyInterface

`Meritum\Serialization\Strategy\StrategyInterface`

Controls the output shape of the [`EnvelopeInterface`](/docs/packages/serialization/api/envelope-interface/) produced by the formatter. Two implementations are provided; register a custom one as `StrategyInterface::class` to override.

## Methods

```php
public function item(ItemInterface $item): EnvelopeInterface;
public function collection(CollectionInterface $collection): EnvelopeInterface;
```

## Built-in implementations

### DataArrayStrategy (default)

`Meritum\Serialization\Strategy\DataArrayStrategy`

Items: serialized fields wrapped in a `"data"` key.

```json
{ "data": { "id": 1, "name": "Jane" } }
```

Collections: serialized items in a `"data"` array, pagination keys at root, optional `"meta"`.

```json
{ "total": 50, "limit": 25, "current": 1, "last": 2, "data": [...] }
```

### ArrayStrategy

`Meritum\Serialization\Strategy\ArrayStrategy`

Items: serialized fields at the root level. Meta appears as a `"meta"` key at root if set.

```json
{ "id": 1, "name": "Jane" }
```

Collections: same shape as `DataArrayStrategy`.

## Registering a strategy

```php
use Meritum\Serialization\Strategy\ArrayStrategy;
use Meritum\Serialization\Strategy\StrategyInterface;

$kernel->define(StrategyInterface::class, fn() => new ArrayStrategy());
```

`SerializationModule` checks for `StrategyInterface::class` in the container when the formatter is first resolved. If present, it is used; otherwise `DataArrayStrategy` is used.
