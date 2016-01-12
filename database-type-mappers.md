# Type Mappers

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Usage](#basic-usage)
3. [Dates](#dates)
  1. [To SQL](#dates-to-sql)
  2. [From SQL](#dates-from-sql)
4. [Times With Time Zones](#times-with-time-zones)
  1. [To SQL](#times-with-time-zones-to-sql)
  2. [From SQL](#times-with-time-zones-from-sql)
5. [Times Without Time Zones](#times-without-time-zones)
  1. [To SQL](#times-without-time-zones-to-sql)
  2. [From SQL](#times-without-time-zones-from-sql)
6. [Timestamps With Time Zones](#timestamps-with-time-zones)
  1. [To SQL](#timestamps-with-time-zones-to-sql)
  2. [From SQL](#timestamps-with-time-zones-from-sql)
7. [Timestamps Without Time Zones](#timestamps-without-time-zones)
  1. [To SQL](#timestamps-without-time-zones-to-sql)
  2. [From SQL](#timestamps-without-time-zones-from-sql)
8. [Booleans](#booleans)
  1. [To SQL](#booleans-to-sql)
  2. [From SQL](#booleans-from-sql)
9. [JSON](#json)
  1. [To SQL](#json-to-sql)
  2. [From SQL](#json-from-sql)

<h2 id="introduction">Introduction</h2>
A common task when reading and writing to a database is translating PHP values into database values and vice versa.  For example, you might store a date as a `DateTime` in PHP, but you need to translate this into a string with a `Y-m-d H:i:s` format before storing it in the database.  *Providers* contain the rules for each database provider (eg MySQL, PostgreSQL, etc) on how to translate PHP and database values.  Using a combination of type mappers and providers, you can translate PHP-to-database and database-to-PHP values.

<h2 id="basic-usage">Basic Usage</h2>
Type mappers need a provider, eg MySQL or PostgreSQL, to do the conversions.  You can either pass the provider in the constructor:

```php
use Opulence\Databases\Providers\MySqlProvider;
use Opulence\Databases\Providers\Types\TypeMapper;

$typeMapper = new TypeMapper(new MySqlProvider());
```

Opulence provides a factory to create type mappers from providers:

```php
use Opulence\Databases\Providers\Types\Factories\TypeMapperFactory;

$factory = new TypeMapperFactory();
// Let's assume $connection is an instance of Opulence\Databases\IConnection
$typeMapper = $factory->create($connection->getDatabaseProvider());
```

All methods accept a provider in the last parameter:

```php
$typeMapper->toSqlTimestampWithTimeZone(new DateTime(), new MySqlProvider());
```

> **Note:** For all of the following examples, the `PostgreSqlProvider` is used.  However, you can use any provider you'd like in your application.

<h2 id="dates">Dates</h2>

<h4 id="dates-to-sql">To SQL</h4>
```php
$phpDate = new DateTime("1987-07-24 12:34:56");
echo $typeMapper->toSqlDate($phpDate); // "1987-07-24"
```

> **Note:** This method accepts any object implementing `DateTimeInterface`, including `DateTimeImmutable`.

<h4 id="dates-from-sql">From SQL</h4>
```php
$sqlDate = "1987-07-24";
$phpDate = $typeMapper->fromSqlDate($sqlDate);
echo $phpDate->format("Y-m-d"); // "1987-07-24"
```

<h2 id="times-with-time-zones">Times With Time Zones</h2>

<h4 id="times-with-time-zones-to-sql">To SQL</h4>
```php
$phpTime = new DateTime("1987-07-24 12:34:56", new DateTimeZone("UTC"));
echo $typeMapper->toSqlTimeWithTimeZone($phpTime); // "12:34:56+0000"
```

> **Note:** This method accepts any object implementing `DateTimeInterface`, including `DateTimeImmutable`.

<h4 id="times-with-time-zones-from-sql">From SQL</h4>
```php
$sqlTime = "12:34:56+0000";
$phpTime = $typeMapper->fromSqlTimeWithTimeZone($sqlTime);
echo $phpTime->format("H:i:sO"); // "12:34:56+0000"
```

<h2 id="times-without-time-zones">Times Without Time Zones</h2>

<h4 id="times-without-time-zones-to-sql">To SQL</h4>
```php
$phpTime = new DateTime("1987-07-24 12:34:56");
echo $typeMapper->toSqlTimeWithoutTimeZone($phpTime); // "12:34:56"
```

> **Note:** This method accepts any object implementing `DateTimeInterface`, including `DateTimeImmutable`.

<h4 id="times-without-time-zones-from-sql">From SQL</h4>
```php
$sqlTime = "12:34:56";
$phpTime = $typeMapper->fromSqlTimeWithoutTimeZone($sqlTime);
echo $phpTime->format("H:i:s"); // "12:34:56"
```

<h2 id="timestamps-with-time-zones">Timestamps With Time Zones</h2>

<h4 id="timestamps-with-time-zones-to-sql">To SQL</h4>
```php
$phpTimestamp = new DateTime("1987-07-24 12:34:56", new DateTimeZone("UTC"));
echo $typeMapper->toSqlTimestampWithTimeZone($phpTimestamp); // "1987-07-24 12:34:56+0000"
```

> **Note:** This method accepts any object implementing `DateTimeInterface`, including `DateTimeImmutable`.

<h4 id="timestamps-with-time-zones-from-sql">From SQL</h4>
```php
$sqlTimestamp = "1987-07-24 12:34:56+0000";
$phpTimestamp = $typeMapper->fromSqlTimestampWithTimeZone($sqlTimestamp);
echo $phpTimestamp->format("Y-m-d H:i:sO"); // "1987-07-24 12:34:56+0000"
```

<h2 id="timestamps-without-time-zones">Timestamps Without Time Zones</h2>

<h4 id="timestamps-without-time-zones-to-sql">To SQL</h4>
```php
$phpTimestamp = new DateTime("1987-07-24 12:34:56");
echo $typeMapper->toSqlTimestampWithoutTimeZone($phpTimestamp); // "1987-07-24 12:34:56"
```

> **Note:** This method accepts any object implementing `DateTimeInterface`, including `DateTimeImmutable`.

<h4 id="timestamps-without-time-zones-from-sql">From SQL</h4>
```php
$sqlTimestamp = "1987-07-24 12:34:56";
$phpTimestamp = $typeMapper->fromSqlTimestampWithoutTimeZone($sqlTimestamp);
echo $phpTimestamp->format("Y-m-d H:i:s"); // "1987-07-24 12:34:56"
```

<h2 id="booleans">Booleans</h2>

<h4 id="booleans-to-sql">To SQL</h4>
```php
$phpBoolean = false;
echo $typeMapper->toSqlBoolean($phpBoolean); // "f"
```

<h4 id="booleans-from-sql">From SQL</h4>
```php
$sqlBoolean = "t";
$phpBoolean = $typeMapper->fromSqlBoolean($sqlBoolean);
echo $phpBoolean === true; // 1
```

<h2 id="json">JSON</h2>

<h4 id="json-to-sql">To SQL</h4>
```php
$phpArray = ["foo" => "bar"];
echo $typeMapper->toSqlJson($phpArray); // '{"foo":"bar"}'
```

<h4 id="json-from-sql">From SQL</h4>
```php
$sqlJson = '{"foo":"bar"}';
$phpArray = $typeMapper->fromSqlJson($sqlJson);
echo $phpArray["foo"]; // "bar"
```