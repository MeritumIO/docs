+++
title = "Blueprint"
description = "Fluent table definition used inside SchemaInterface create() and alter() callbacks"
weight = 4
+++

# Blueprint

`Georgeff\Schema\Blueprint`

Passed to the callback in `SchemaInterface::create()` and `SchemaInterface::alter()`. Defines columns, indexes, and foreign keys to add, and columns/indexes/foreign keys to drop.

All column methods return a [`Column`](/docs/packages/migrations/api/column/) for fluent modifier chaining. `foreign()` returns a [`ForeignKey`](/docs/packages/migrations/api/foreign-key/).

## Column Methods

### Strings

| Method | SQL type |
|---|---|
| `string(string $name, int $length = 255)` | `VARCHAR($length)` |
| `char(string $name, int $length = 1)` | `CHAR($length)` |
| `text(string $name)` | `TEXT` |

### Integers

| Method | SQL type |
|---|---|
| `id(string $name = 'id')` | `BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY` |
| `integer(string $name)` | `INT` |
| `tinyInteger(string $name)` | `TINYINT` |
| `smallInteger(string $name)` | `SMALLINT` |
| `bigInteger(string $name)` | `BIGINT` |

### Numeric

| Method | SQL type |
|---|---|
| `float(string $name)` | `REAL` |
| `decimal(string $name, int $precision = 8, int $scale = 2)` | `DECIMAL($precision, $scale)` |

### Other types

| Method | SQL type |
|---|---|
| `boolean(string $name)` | `BOOLEAN` |
| `json(string $name)` | `JSON` |
| `uuid(string $name)` | `UUID` / `CHAR(36)` |
| `date(string $name)` | `DATE` |
| `time(string $name)` | `TIME` |
| `timestamp(string $name)` | `TIMESTAMP` |

### Shortcuts

```php
public function timestamps(): void;
```

Adds nullable `created_at` and `updated_at` timestamp columns.

## Foreign Keys

```php
public function foreign(string $column, ?string $name = null): ForeignKey;
```

Adds a foreign key constraint on `$column`. `$name` sets an explicit constraint name; one is generated automatically when omitted. See [`ForeignKey`](/docs/packages/migrations/api/foreign-key/) for the full fluent API.

```php
$blueprint->foreign('user_id')->references('id')->on('users')->onDelete('CASCADE');
```

## Drop Methods (alter only)

```php
public function dropColumn(string $name): self;
public function dropIndex(string $name): self;
public function dropForeign(string $name): self;
```

Queue a column, index, or foreign key constraint for removal. Pass the constraint name to `dropIndex()` and `dropForeign()`.

```php
$schema->alter('posts', function (Blueprint $blueprint) {
    $blueprint->dropColumn('legacy_field');
    $blueprint->dropForeign('posts_author_id_foreign');
});
```
