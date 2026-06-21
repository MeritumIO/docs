+++
title = "Repository"
description = "Abstract base class for model persistence and querying"
weight = 2
+++

# Repository

`Meritum\Database\Repository<T extends Model>`

Abstract base class for all model repositories. Implements [`RepositoryInterface`](/docs/packages/database/api/repository-interface/) and adds protected query-building infrastructure: scopes, pagination, and cursor pagination. Extend it and implement `getModelClass()`.

## Constructor

```php
public function __construct(DatabaseManagerInterface $db);
```

Inject `DatabaseManagerInterface` from the container:

```php
$kernel->define(UserRepository::class, fn($c) => new UserRepository(
    $c->get(DatabaseManagerInterface::class)
));
```

## Abstract Methods

```php
abstract protected function getModelClass(): string;
```

Return the fully-qualified class name of the model this repository manages:

```php
protected function getModelClass(): string
{
    return User::class;
}
```

## Public Persistence Methods

```php
public function save(Model $model): bool;
```

Inserts or updates `$model`. Inserts when `$model->isNew()`. Auto-generates a UUID primary key for new string-keyed models that have no primary key set. Sets `created_at` and `updated_at` on insert; sets only `updated_at` on update. Calls `syncOriginal()` after a successful write.

---

```php
public function delete(Model $model): bool;
```

Deletes `$model` by its primary key. Returns `true` on success.

## Public Finder Methods

```php
public function find(int|string $id): ?T;
```

Finds a model by primary key. Returns `null` when not found.

---

```php
public function findOrFail(int|string $id): T;
```

Finds a model by primary key. Throws `ModelNotFoundException` when not found.

---

```php
public function findBy(string $column, mixed $value): ?T;
```

Finds the first model where `$column` equals `$value`. Returns `null` when not found.

## Protected Query Methods

These methods are intended for use inside subclass query methods.

```php
protected function query(): SelectInterface;
```

Returns a fresh `SelectInterface` scoped to the model's table and primary key. Calling `query()` resets the internal query state — call it at the start of every new query chain, not mid-chain.

---

```php
protected function first(): ?T;
```

Executes the current query and returns the first result as a hydrated model, or `null`.

---

```php
protected function get(): Collection<T>;
```

Executes the current query and returns all results as a typed `Collection`.

---

```php
protected function count(): int;
```

Executes the current query and returns the row count.

---

```php
protected function paginate(int $perPage, int $currentPage): Paginator<T>;
```

Executes the current query with limit/offset and returns a `Paginator`.

---

```php
protected function cursor(int $perPage, ?string $cursor): CursorPaginator<T>;
```

Executes the current query using keyset pagination on the primary key. Pass `null` for the first page; pass the `nextCursor` or `previousCursor` from a previous result for subsequent pages.

This method appends its own `ORDER BY` on the primary key and does not clear any ordering already present on the query. Any prior `orderBy()` call — including those applied by scopes — will stack and produce incorrect results. Call `resetOrderBy()` before invoking `cursor()` if the query or a scope has set an ordering.

UUIDv4 primary keys are randomly generated and have no natural ordering, making them unsuitable as a cursor column. Override `generateUuid()` to return `Uuid::v7()` on any model used with cursor pagination.

## Scope Methods

```php
protected function addScope(string $name, callable $scope): void;
```

Registers a named scope applied automatically to every query. The callable receives the `SelectInterface`:

```php
$this->addScope('active', fn(SelectInterface $q) => $q->where('is_active', true));
```

---

```php
protected function withoutScope(string $name): static;
```

Returns `$this` with the named scope suspended for the next query execution. The scope is re-enabled after execution.

---

```php
protected function withoutScopes(): static;
```

Returns `$this` with all scopes suspended for the next query execution.

## Overridable Methods

```php
protected function generateUuid(): string;
```

Override to change the UUID generator used for new string-keyed models. Defaults to `Uuid::v4()`. Override to `Uuid::v7()` for models used with cursor pagination, as UUIDv4 is randomly ordered and unsuitable as a cursor column:

```php
use Meritum\Database\Support\Uuid;

protected function generateUuid(): string
{
    return Uuid::v7();
}
```
