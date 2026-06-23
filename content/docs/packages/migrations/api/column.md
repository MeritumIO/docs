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
public function defaultRaw(string $sql): self;
```

Sets a default value using a raw SQL expression. The string is emitted verbatim — no quoting is applied. Use this for database functions and other expressions that cannot be represented as a PHP literal.

```php
$blueprint->timestamp('created_at')->defaultRaw('CURRENT_TIMESTAMP');
$blueprint->timestamp('updated_at')->defaultRaw('CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP');
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
    $blueprint->timestamp('created_at')->defaultRaw('CURRENT_TIMESTAMP');
});
```
