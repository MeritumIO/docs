+++
title = "MigrationInterface"
description = "Contract for a single migration — up and down"
weight = 1
+++

# MigrationInterface

`Meritum\Migrations\MigrationInterface`

The contract every migration must implement. Each migration file returns an anonymous class that implements this interface.

## Methods

```php
public function up(SchemaInterface $schema): void;
```

Applied when the migration is run. Use `$schema` to create or alter tables.

---

```php
public function down(SchemaInterface $schema): void;
```

Applied when the migration is rolled back. Should reverse everything done in `up()`.

## Example

```php
<?php

use Georgeff\Schema\Blueprint;
use Meritum\Migrations\SchemaInterface;
use Meritum\Migrations\MigrationInterface;

return new class implements MigrationInterface {

    public function up(SchemaInterface $schema): void
    {
        $schema->create('posts', function (Blueprint $blueprint) {
            $blueprint->uuid('id')->unique()->primary();
            $blueprint->string('title');
            $blueprint->text('body');
            $blueprint->timestamps();
        });
    }

    public function down(SchemaInterface $schema): void
    {
        $schema->dropIfExists('posts');
    }
};
```
