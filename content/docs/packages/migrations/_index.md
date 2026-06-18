+++
bookCollapseSection = true
weight = 3
title = "meritum/migrations"
description = "Schema migration runner with a CLI interface for the Meritum ecosystem"
+++

# meritum/migrations

`meritum/migrations` is a schema migration runner with a dedicated CLI. It ships as a standalone binary (`bin/strata`) that you can run directly, and as a pair of modules you can add to any existing `meritum/cli` kernel.

```
composer require meritum/migrations
```

## Standalone Usage

After installation, run migrations directly through the installed binary:

```bash
vendor/bin/strata migration:install
vendor/bin/strata migration:run
```

Configure the database and migration path entirely through environment variables — no code required.

### Configuration

| Variable | Default | Description |
|---|---|---|
| `DB_DRIVER` | `pgsql` | Database driver: `pgsql`, `mysql`, or `sqlite` |
| `DB_HOST` | `localhost` | Database host |
| `DB_PORT` | `5432` | Database port |
| `DB_DATABASE` | — | Database name |
| `DB_USERNAME` | — | Database username |
| `DB_PASSWORD` | — | Database password |
| `DB_PGSQL_SCHEMA` | `public` | PostgreSQL schema |
| `DB_PGSQL_SSL_MODE` | `prefer` | PostgreSQL SSL mode |
| `DB_MYSQL_CHARSET` | `utf8mb4` | MySQL character set |
| `DB_MIGRATIONS_TABLE` | `migrations` | Table used to track ran migrations |
| `DB_MIGRATIONS_PATH` | `{cwd}/migrations` | Directory containing migration files |

## Integration with an Existing Kernel

If you already have a `meritum/cli` kernel, add the migrations modules to your module repository instead of using the standalone binary.

If your kernel already registers `meritum/database`'s `DatabaseModule`, only add `MigrationsModule` — it registers on top of the existing database services:

```php
use Meritum\Migrations\Module\MigrationsModule;

final class ModuleRepository implements ModuleRepositoryInterface
{
    public function modules(Environment $env): array
    {
        return [
            new DatabaseModule(),
            new MigrationsModule(),
            // your other modules...
        ];
    }
}
```

If your kernel does not already have database services registered, add both migration modules (they bundle their own):

```php
use Meritum\Migrations\Module\DatabaseModule as MigrationsDbModule;
use Meritum\Migrations\Module\MigrationsModule;

return [
    new MigrationsDbModule(),
    new MigrationsModule(),
    // ...
];
```

`MigrationsModule` tags all five migration commands with `'cli.commands'` — the CLI kernel picks them up automatically.

## Writing Migrations

Generate a migration file with `migration:make`:

```bash
# Blank migration
vendor/bin/strata migration:make create_users

# Create-table stub (pre-filled with uuid + timestamps)
vendor/bin/strata migration:make create_users --create-table=users

# Alter-table stub
vendor/bin/strata migration:make add_role_to_users --alter-table=users
```

`--create-table` and `--alter-table` are mutually exclusive — passing both in the same call is an error.

Files are written to `DB_MIGRATIONS_PATH` (default `migrations/`) with a timestamp prefix:

```
migrations/
  2026_06_17_153045_create_users.php
```

Each migration is an anonymous class returned from the file:

```php
<?php

use Georgeff\Schema\Blueprint;
use Meritum\Migrations\SchemaInterface;
use Meritum\Migrations\MigrationInterface;

return new class implements MigrationInterface {

    public function up(SchemaInterface $schema): void
    {
        $schema->create('users', function (Blueprint $blueprint) {
            $blueprint->uuid('id')->unique()->primary();
            $blueprint->string('name');
            $blueprint->string('email')->unique();
            $blueprint->boolean('is_active')->default(true);
            $blueprint->timestamps();
        });
    }

    public function down(SchemaInterface $schema): void
    {
        $schema->dropIfExists('users');
    }
};
```

## Schema API

`SchemaInterface` is injected into every `up()` and `down()` method:

| Method | Description |
|---|---|
| `create(string $table, callable $callback)` | Create a new table. The callback receives a `Blueprint`. |
| `alter(string $table, callable $callback)` | Modify an existing table. The callback receives a `Blueprint`. |
| `drop(string $table)` | Drop a table. |
| `dropIfExists(string $table)` | Drop a table if it exists. |
| `tableExists(string $table)` | Returns `true` if the table exists. |

## Commands

| Command | Description | Flags |
|---|---|---|
| `migration:install` | Create the migrations tracking table | `--force / -f` |
| `migration:run` | Run all pending migrations | `--force / -f` |
| `migration:rollback` | Roll back the last batch of migrations | `--force / -f` |
| `migration:status` | Display migration status table | — |
| `migration:make <filename>` | Generate a migration file | `--create-table=`, `--alter-table=` |

`--force / -f` is required when `APP_ENV=production` for install, run, and rollback.

## Batches and Rollback

Each `migration:run` call increments the batch number and tags every migration run in that call with the same batch. `migration:rollback` reverses only the migrations in the most recent batch, in reverse order. Running `migration:rollback` again rolls back the next most recent batch.

```bash
# Run pending migrations (batch 1)
vendor/bin/strata migration:run

# Roll back batch 1
vendor/bin/strata migration:rollback

# Check what has and hasn't been run
vendor/bin/strata migration:status
```
