+++
title = "ForeignKey"
description = "Fluent foreign key constraint returned by Blueprint::foreign()"
weight = 6
+++

# ForeignKey

`Georgeff\Schema\ForeignKey`

Returned by `Blueprint::foreign()`. All methods return `self` for chaining. `onDelete` and `onUpdate` default to `RESTRICT`.

## Methods

```php
public function references(string $column): self;
```

The column on the referenced table.

---

```php
public function on(string $table): self;
```

The referenced table.

---

```php
public function onDelete(string $action): self;
```

The referential action on delete. Valid values: `NO ACTION`, `RESTRICT`, `SET NULL`, `CASCADE`. Case-insensitive. Throws `InvalidArgumentException` for unrecognised actions.

---

```php
public function onUpdate(string $action): self;
```

The referential action on update. Same valid values as `onDelete`.

## Example

```php
$schema->create('posts', function (Blueprint $blueprint) {
    $blueprint->uuid('id')->unique()->primary();
    $blueprint->uuid('user_id');
    $blueprint->string('title');
    $blueprint->timestamps();

    $blueprint->foreign('user_id')
              ->references('id')
              ->on('users')
              ->onDelete('CASCADE')
              ->onUpdate('RESTRICT');
});
```
