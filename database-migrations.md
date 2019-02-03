# Database Migrations

## Table of Contents
1. [Introduction](#introduction)
2. [Creating Migrations](#creating-migrations)
3. [Migrator](#migrator)
4. [Running Migrations](#running-migrations)
  1. [Logging Migrations](#logging-migrations)
5. [Rolling Back Migrations](#rolling-back-migrations)

<h2 id="introduction">Introduction</h2>

As your product changes, chances are you'll need to make changes to the structure of your database tables.  Migrations give you the ability to make these changes atomically, and roll them back in case of an issue.

<h2 id="creating-migrations">Creating Migrations</h2>

Migrations implement `Opulence\Databases\Migrations\IMigration` (`Migration` comes built-in).  It defines two methods:

* `down()`
  * Performs a rollback of the migration
* `up()`
  * Runs the migration

To create a migration, you can run

```php
php apex make:migration MIGRATION_NAME
```

This will generate a file in _src/Infrastructure/Databases/Migrations_.  For example, running 

```php
php apex make:migration AddUserTable
```

will generate a class like the following:

```php
class AddUserTable extends Migration
{
    public static function getCreationDate() : DateTime
    {
        return DateTime::createFromFormat(DateTime::ATOM, '2017-01-01T12:00:00+00:00');
    }

    public function down() : void
    {
        // 
    }

    public function up() : void
    {
        // 
    }
}
```

> **Note:** `getCreationDate()` is used to arrange the migrations in the order they'll be run.

`$this->connection` contains an instance of `Opulence\Databases\IConnection`.  You can use this to execute your migration:

```php
public function down() : void
{
    $sql = 'DROP TABLE IF EXISTS users';
    $statement = $this->connection->prepare($sql);
    $statement->execute();
}

public function up() : void
{
    $sql = 'CREATE TABLE users (id serial primary key, email text)';
    $statement = $this->connection->prepare($sql);
    $statement->execute();
}
```

> **Note:** There is no need to create database transactions in your migrations - one is created automatically by the migrator.

<h2 id="migrator">Migrator</h2>

Migrations are discovered, run, and rolled back by an instance of `Opulence\Databases\Migrations\IMigrator` (`Migrator` comes built-in).  Migration classes are discovered by `FileMigrationFinder`, which recursively finds all classes that implement `IMigration` in a particular path or paths.  Those migrations are ordered by their creation dates.

If you're using the <a href="https://github.com/opulencephp/Project" target="_blank">skeleton project</a>, you can configure the path to search for migration classes in _config/paths.php_ (defaults to _src/Infrastructure/Databases/Migrations_).

<h2 id="running-migrations">Running Migrations</h2>

To run migrations, call

```php
php apex migrations:up
```

This will call the `up()` methods on any un-executed migrations.  If you'd like to run them outside of the console, you can call `IMigrator::runMigrations()`.

<h3 id="logging-migrations">Logging Migrations</h3>

Once you run a migration, it is added to some sort of storage - typically a database.  This is done via `IExecutedMigrationRepository` (`SqlExecutedMigrationRepository` comes built-in if you're using the skeleton project).

> **Note:** Migrations are keyed by name in the database.  If you ever want to re-namespace your migrations or change their class names, be sure to run a migration beforehand updating the names of the migrations in the database.

<h2 id="rolling-back-migrations">Rolling Back Migrations</h2>

Sometimes, migrations don't go quite as planned, and it'd be nice to roll them back.  You can do so using

```php
php apex migrations:down --number NUMBER_TO_ROLL_BACK
```

This will call all the `down()` methods in your executed migrations.  If you'd like to roll back all your migrations, simply don't pass the `--number` option.  If you'd like to roll back migrations outside of the console, you can call `IMigrator::rollBackMigrations()`.

<h2 id="using-mysql">Using MySQL</h2>

The migrator will use the PostgreSQL database adapter by default. If you need MySQL configured, you need ensure that the `DB_DRIVER` environment variable is set correctly. If you're using `opulence/project`, then the relevant part setting is in `config/environment/.env.app.php`:

```php
Environment::setVar('DB_DRIVER', \Opulence\Databases\Adapters\Pdo\MySql\Driver::class);
```

