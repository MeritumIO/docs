+++
title = "Model"
description = "Abstract base class for database-backed entities"
weight = 1
+++

# Model

`Meritum\Database\Model`

Abstract base class for all database-backed entities. Provides attribute storage, cast conversion, dirty tracking, and PHP 8.4 property hook integration.

## Configuration Properties

Declare these as `protected` properties in your subclass:

```php
protected string $table;             // required — the database table name
protected string $primaryKey     = 'id';
protected string $primaryKeyType = 'string'; // 'string' or 'int'
protected bool   $incrementing   = false;
protected bool   $timestamps     = true;
protected string $createdAtColumn = 'created_at';
protected string $updatedAtColumn = 'updated_at';
protected array  $casts          = [];
```

## Property Hooks

Access attributes through PHP 8.4 property hooks backed by `getAttribute()` and `setAttribute()`:

```php
public string $name {
    get => $this->getAttribute('name');
    set => $this->setAttribute('name', $value);
}
```

Read-only attributes omit the `set` hook. Cast values are applied automatically when the column name has an entry in `$casts`.

## Protected Methods

```php
protected function getAttribute(string $key): mixed;
```

Returns the attribute value for `$key`, applying any registered accessor and cast.

---

```php
protected function setAttribute(string $key, mixed $value): void;
```

Stores `$value` for `$key`, applying any registered mutator before storage.

---

```php
protected function accessors(): array;
```

Override to register per-column read transformations. Return an array keyed by column name:

```php
protected function accessors(): array
{
    return [
        'name' => fn(string $v) => ucwords($v),
    ];
}
```

---

```php
protected function mutators(): array;
```

Override to register per-column write transformations. Return an array keyed by column name:

```php
protected function mutators(): array
{
    return [
        'email' => fn(string $v) => strtolower($v),
    ];
}
```

## Public Methods

```php
public function isNew(): bool;
```

Returns `true` if the model has not been persisted (has no original attributes).

---

```php
public function isDirty(): bool;
```

Returns `true` if any attribute has changed since the model was last synced.

---

```php
public function hydrate(array $attributes): void;
```

Populates the model from a raw database row. Throws `LogicException` if the model already has attributes (call `hydrate()` only on fresh instances).

---

```php
public function toArray(): array;
```

Returns all attributes as a plain array. `DateTimeImmutable` values are formatted as ISO 8601 strings. Any loaded relations are merged in after attributes; `JsonSerializable` relation objects have `jsonSerialize()` called, scalars and arrays are included as-is.

---

```php
protected function setRelation(string $name, mixed $relation): void;
```

Stores a loaded relation in the relation bag under `$name`. Throws `LogicException` if `$relation` is `null` or an object that does not implement `JsonSerializable`.

---

```php
protected function getRelation(string $name): mixed;
```

Returns the stored relation for `$name`, or `null` if it has not been set.

---

```php
protected function hasRelation(string $name): bool;
```

Returns `true` if a relation has been stored under `$name`.

## Serialization

`Model` implements PHP native serialization via `__serialize()`/`__unserialize()`. Attributes and relations are persisted; the model has no dirty state on unserialize. This makes models compatible with any cache backend that relies on `serialize()` (PSR-6, PSR-16, Redis, APCu).

```php
$user = $repository->findOrFail($id);
$cache->set("user:{$id}", serialize($user));

$user = unserialize($cache->get("user:{$id}"));
```

Callers are responsible for ensuring any relations stored on the model are themselves serializable.
