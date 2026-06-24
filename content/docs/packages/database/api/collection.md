+++
title = "Collection"
description = "Typed, iterable collection of models keyed by primary key"
weight = 3
+++

# Collection

`Meritum\Database\Support\Collection<T extends Model>`

An ordered, typed collection of models keyed by their primary key. Implements `Iterator`, `Countable`, and `JsonSerializable`. Returned by `Repository::get()` and by `Paginator::collection()` / `CursorPaginator::collection()`.

## Methods

```php
public function isEmpty(): bool;
public function isNotEmpty(): bool;
```

---

```php
public function has(int|string $key): bool;
```

Returns `true` if the collection contains a model with the given primary key.

---

```php
public function get(int|string $key): ?T;
```

Returns the model with the given primary key, or `null`.

---

```php
public function all(): array<int|string, T>;
```

Returns all models as an associative array keyed by primary key.

---

```php
public function keys(): array<int, int|string>;
```

Returns all primary key values as a list.

---

```php
public function first(): ?T;
```

Returns the first model in the collection, or `null` if empty.

---

```php
public function last(): ?T;
```

Returns the last model in the collection, or `null` if empty.

---

```php
public function filter(callable $predicate): static;
```

Returns a new collection containing only models for which `$predicate` returns `true`:

```php
$active = $users->filter(fn(User $u) => $u->isActive);
```

---

```php
public function each(callable $callback): void;
```

Calls `$callback` with each model in iteration order. Does not return a value.

---

```php
public function push(T $model): static;
```

Returns a new collection with `$model` appended.

---

```php
public function merge(Collection<T> $other): static;
```

Returns a new collection containing all models from both collections. Models in `$other` overwrite models with the same primary key.

---

```php
public function count(): int;
```

Returns the number of models in the collection.

---

```php
public function toArray(): array;
```

Returns all models serialized as arrays. Each model's `toArray()` is called, producing a list of attribute arrays.

---

```php
public function jsonSerialize(): array;
```

Same as `toArray()`. Allows `json_encode($collection)` to work directly.

## Serialization

`Collection` supports PHP native serialization via `__serialize()`/`__unserialize()`. Model keys are preserved and each model's own serialization is used, making collections compatible with any cache backend that relies on `serialize()`.

```php
$users = $repository->findActive();
$cache->set('active_users', serialize($users));

$users = unserialize($cache->get('active_users'));
```
