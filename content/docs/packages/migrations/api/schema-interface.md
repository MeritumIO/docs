+++
title = "SchemaInterface"
description = "DDL operations injected into migration up() and down() methods"
weight = 2
+++

# SchemaInterface

`Meritum\Migrations\SchemaInterface`

Provides DDL operations over the write connection. Injected by the migrator into every `up()` and `down()` call. The concrete implementation selects SQL generation strategy based on `DB_DRIVER`.

## Methods

```php
public function create(string $table, callable $callback): void;
```

Creates a new table. The callback receives a `Blueprint` to define columns and indexes:

```php
$schema->create('users', function (Blueprint $blueprint) {
    $blueprint->uuid('id')->unique()->primary();
    $blueprint->string('email')->unique();
    $blueprint->timestamps();
});
```

---

```php
public function alter(string $table, callable $callback): void;
```

Modifies an existing table. The callback receives a `Blueprint`:

```php
$schema->alter('users', function (Blueprint $blueprint) {
    $blueprint->string('name');
});
```

---

```php
public function drop(string $table): void;
```

Drops the table unconditionally.

---

```php
public function dropIfExists(string $table): void;
```

Drops the table only if it exists. Safe to use in `down()` methods.

---

```php
public function tableExists(string $table): bool;
```

Returns `true` if the table exists in the current database. Useful for conditional logic inside migrations.
