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
A common task when reading and writing to a database is translating PHP values into database values and vice versa.  For example, you might store a date as a `DateTime` in PHP, but you need to translate this into a string with a `Y-m-d H:i:s` format before storing it in the database.  *Providers* contain the rules for each database provider (eg MySQL, PostgreSQL, etc) on how to translate PHP and database values.  Using a combination of *type mappers* and *providers*, you can translate PHP-to-database and database-to-PHP values.

<h2 id="basic-usage">Basic Usage</h2>
Type mappers need a provider, eg MySQL or PostgreSQL, to do the conversions.  You can either pass the provider in the constructor:

```php
use RDev\Databases\Providers\MySQLProvider;
use RDev\Databases\Providers\TypeMapper;

$typeMapper = new TypeMapper(new MySQLProvider());
```

You can also use `setProvider()`.  Alternatively, all methods accept a provider in the last parameter:

```php
$typeMapper->toSQLTimestampWithTimeZone(new \DateTime(), new MySQLProvider());
```

> **Note:** For all of the following examples, the `PostgreSQLProvider` is used.  However, you can use any provider you'd like in your application.

<h2 id="dates">Dates</h2>

<h4 id="dates-to-sql">To SQL</h4>
```php
$phpDate = new DateTime("1987-07-24 12:34:56");
echo $typeMapper->toSQLDate($phpDate); // "1987-07-24"
```

<h4 id="dates-from-sql">From SQL</h4>
```php
$sqlDate = "1987-07-24";
$phpDate = $typeMapper->fromSQLDate($sqlDate);
echo $phpDate->format("Y-m-d"); // "1987-07-24"
```

<h2 id="times-with-time-zones">Times With Time Zones</h2>

<h4 id="times-with-time-zones-to-sql">To SQL</h4>
```php
$phpTime = new DateTime("1987-07-24 12:34:56", new DateTimeZone("UTC"));
echo $typeMapper->toSQLTimeWithTimeZone($phpTime); // "12:34:56+0000"
```

<h4 id="times-with-time-zones-from-sql">From SQL</h4>
```php
$sqlTime = "12:34:56+0000";
$phpTime = $typeMapper->fromSQLTimeWithTimeZone($sqlTime);
echo $phpTime->format("H:i:sO"); // "12:34:56+0000"
```

<h2 id="times-without-time-zones">Times Without Time Zones</h2>

<h4 id="times-without-time-zones-to-sql">To SQL</h4>
```php
$phpTime = new DateTime("1987-07-24 12:34:56");
echo $typeMapper->toSQLTimeWithoutTimeZone($phpTime); // "12:34:56"
```

<h4 id="times-without-time-zones-from-sql">From SQL</h4>
```php
$sqlTime = "12:34:56";
$phpTime = $typeMapper->fromSQLTimeWithoutTimeZone($sqlTime);
echo $phpTime->format("H:i:s"); // "12:34:56"
```

<h2 id="timestamps-with-time-zones">Timestamps With Time Zones</h2>

<h4 id="timestamps-with-time-zones-to-sql">To SQL</h4>
```php
$phpTimestamp = new DateTime("1987-07-24 12:34:56", new DateTimeZone("UTC"));
echo $typeMapper->toSQLTimestampWithTimeZone($phpTimestamp); // "1987-07-24 12:34:56+0000"
```

<h4 id="timestamps-with-time-zones-from-sql">From SQL</h4>
```php
$sqlTimestamp = "1987-07-24 12:34:56+0000";
$phpTimestamp = $typeMapper->fromSQLTimestampWithTimeZone($sqlTimestamp);
echo $phpTimestamp->format("Y-m-d H:i:sO"); // "1987-07-24 12:34:56+0000"
```

<h2 id="timestamps-without-time-zones">Timestamps Without Time Zones</h2>

<h4 id="timestamps-without-time-zones-to-sql">To SQL</h4>
```php
$phpTimestamp = new DateTime("1987-07-24 12:34:56");
echo $typeMapper->toSQLTimestampWithoutTimeZone($phpTimestamp); // "1987-07-24 12:34:56"
```

<h4 id="timestamps-without-time-zones-from-sql">From SQL</h4>
```php
$sqlTimestamp = "1987-07-24 12:34:56";
$phpTimestamp = $typeMapper->fromSQLTimestampWithoutTimeZone($sqlTimestamp);
echo $phpTimestamp->format("Y-m-d H:i:s"); // "1987-07-24 12:34:56"
```

<h2 id="booleans">Booleans</h2>

<h4 id="booleans-to-sql">To SQL</h4>
```php
$phpBoolean = false;
echo $typeMapper->toSQLBoolean($phpBoolean); // "f"
```

<h4 id="booleans-from-sql">From SQL</h4>
```php
$sqlBoolean = "t";
$phpBoolean = $typeMapper->fromSQLBoolean($sqlBoolean);
echo $phpBoolean === true; // 1
```

<h2 id="json">JSON</h2>

<h4 id="json-to-sql">To SQL</h4>
```php
$phpArray = ["foo" => "bar"];
echo $typeMapper->toSQLJSON($phpArray); // '{"foo":"bar"}'
```

<h4 id="json-from-sql">From SQL</h4>
```php
$sqlJSON = '{"foo":"bar"}';
$phpArray = $typeMapper->fromSQLJSON($sqlJSON);
echo $phpArray["foo"]; // "bar"
```