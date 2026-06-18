+++
title = "Column"
description = "Fluent column modifier returned by Blueprint column methods"
weight = 5
+++

# Column

`Georgeff\Schema\Column`

Returned by every column method on [`Blueprint`](/docs/packages/migrations/api/blueprint/). All modifier methods return `self` for chaining.

## Modifiers

```php
public function nullable(bool $value = true): self;
```

Marks the column as `NULL`. Call `nullable(false)` to explicitly set `NOT NULL`.

---

```php
public function default(mixed $value = null): self;
```

Sets a default value. Pass `null` to default to SQL `NULL`.

```php
$blueprint->boolean('is_active')->default(true);
$blueprint->string('status')->default('pending');
```

---

```php
public function unsigned(): self;
```

Marks an integer column as `UNSIGNED`.

---

```php
public function primary(?string $name = null): self;
```

Adds a primary key index on this column. `$name` sets an explicit index name.

---

```php
public function unique(?string $name = null): self;
```

Adds a unique index on this column. `$name` sets an explicit index name.

---

```php
public function index(?string $name = null): self;
```

Adds a plain index on this column. `$name` sets an explicit index name.

## Example

```php
$schema->create('products', function (Blueprint $blueprint) {
    $blueprint->uuid('id')->unique()->primary();
    $blueprint->string('sku', 64)->unique();
    $blueprint->string('name');
    $blueprint->decimal('price', 10, 2)->unsigned();
    $blueprint->integer('stock')->default(0)->unsigned();
    $blueprint->boolean('is_available')->default(false);
    $blueprint->timestamps();
});
```
