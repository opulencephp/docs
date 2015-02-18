# Query Builders

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Usage](#basic-usage)
3. [Clauses](#clauses)
4. [Binding Values](#binding-values)
5. [Select Queries](#select-queries)
6. [Insert Queries](#insert-queries)
7. [Update Queries](#update-queries)
8. [Using Query Builders with PDO](#using-query-builders-with-pdo)
9. [Database-Specific Query Builders](#database-specific-query-builders)

<h2 id="introduction">Introduction</h2>
Sometimes you need to programmatically generate SQL queries.  Rather than concatenating strings together, you can use `Query Builders` to do the heavy lifting.  They provide a fluent syntax for creating queries and binding values for use in `PDOStatement` or [RDev's PDO wrapper](rdbms).  They even support vendor-specific query features, such as MySQL's `LIMIT` clause support for `DELETE` statements.

<h2 id="basic-usage">Basic Usage</h2>
Let's look at a simple `SELECT` query:

```php
use RDev\Databases\SQL\QueryBuilders\PostgreSQL;

$query = (new PostgreSQL\QueryBuilder)->select("id", "name", "email")
    ->from("users")
    ->where("datejoined < NOW()");
echo $query->getSQL();
```

This will output:
```
SELECT id, name, email FROM users WHERE datejoined < NOW()
```

<h2 id="Clauses">Clauses</h2>
`QueryBuilders` support a variety of clauses.  You may use the following clauses to build complex, but easy-to-read-and-maintain queries:

* FROM
  * `from($tableName, $tableAlias)`
* WHERE
  * `where($condition)`
  * `andWhere($condition)`
  * `orWhere($condition)`
* JOIN
  * `join($tableName, $tableAlias, $condition)`
  * `innerJoin($tableName, $tableAlias, $condition)`
  * `leftJoin($tableName, $tableAlias, $condition)`
  * `rightJoin($tableName, $tableAlias, $condition)`
* GROUP
  * `groupBy($expression)`
  * `addGroupBy($expression)`
* HAVING
  * `having($condition)`
  * `andHaving($condition)`
  * `orHaving($condition)`
* ORDER BY
  * `orderBy($condition)`
  * `addOrderBy($expression)`
* LIMIT
  * `limit($numRows)`
  * `offset($numRows)`
* RETURNING (PostgreSQL only)
  * `returning($expression)`
  * `addReturning($expression)`

<h2 id="binding-values">Binding Values</h2>
`QueryBuilders` provide an intuitive syntax for binding values to queries ([learn more about statement bindings](rdbms)).  To add a named placeholder, use `addNamedPlaceholderValue()`:

```php
$query = (new PostgreSQL\QueryBuilder)->select("content")
    ->from("posts")
    ->where("id < :id")
    ->addNamedPlaceholderValue("id", 24, \PDO::PARAM_INT);
```

To add many named placeholder values, use `addNamedPlaceholderValues()`:
 
```php
$query = (new PostgreSQL\QueryBuilder)->select("count(*)")
    ->from("users")
    ->where("username = :username")
    ->orWhere("id = :id")
    ->addNamedPlaceholderValues([
        // Non-array values are assumed to be of type PDO::PARAM_STR
        "username" => "dave_y",
        // In array values, the first item is the value, and the second is the parameter type
        "id" => [24, \PDO::PARAM_INT]
    ]);
```

Similarly, `addUnnamedPlaceholderValue()` and `addUnnamedPlaceholderValues()` can be used to add unnamed placeholder values.

> **Note:** You cannot mix named with unnamed placeholders.  Also, if no type is specified for a bound value, it's assumed to be PDO::PARAM_STR.

<h2 id="select-queries">Select Queries</h2>
Select queries use a variable argument list to specify the columns to select:

```php
$query = (new PostgreSQL\QueryBuilder)->select("title", "author")
    ->from("books");
echo $query->getSQL();
```

This will output:

```
SELECT title, author FROM books
```

<h2 id="insert-queries">Insert Queries</h2>
Insert queries accept a table name and a mapping of column names to values:

```php
$query = (new PostgreSQL\QueryBuilder)->insert("users", [
    "name" => ":name",
    "email" => ":email",
    "datejoined" => "NOW()"
]);
echo $query->getSQL();
```

This will output:

```
INSERT INTO users (name, email, datejoined) VALUES (:name, :email, NOW())
```

<h2 id="update-queries">Update Queries</h2>
Update queries accept a table name, table alias, and a mapping of column names to values:

```php
$query = (new PostgreSQL\QueryBuilder)->update("users", "u", ["name" => ":name"])
    ->where("id = :id");
echo $query->getSQL();
```

This will output:

```
UPDATE users AS u SET name = :name WHERE id = :id
```

<h2 id="delete-queries">Delete Queries</h2>
Delete queries accept a table name:

```php
$query = (new PostgreSQL\QueryBuilder)->delete("users")
    ->where("id = :id");
echo $query->getSQL();
```

This will output:

```
DELETE FROM users WHERE id = :id
```

<h2 id="using-query-builders-with-pdo">Using Query Builders with PDO</h2>
Let's say you've built the following query:

```php
$query = (new PostgreSQL\QueryBuilder)->select("author")
    ->from("books")
    ->where("title = :title")
    ->addNamedPlaceholderValue("title", "Code Complete");
```

Simply call `getSQL()` and `getParameters()` to use this in `PDO` or in [RDev's PDO wrapper](rdbms):

```php
$statement = $connection->prepare($query->getSQL());
$statement->bindValues($query->getParameters());
$statement->execute();
```

<h2 id="database-specific-query-builders">Database-Specific Query Builders</h2>
MySQL and PostgreSQL have their own query builders, which implement features that are unique to each database.  For example, the MySQL query builder supports a *LIMIT* clause:

```php
use RDev\Databases\SQL\QueryBuilders\MySQL;

$deleteQuery = (new MySQL\QueryBuilder)->delete->delete("users")
    ->where("name = 'dave'")
    ->limit(1);    
echo $deleteQuery->getSQL();
```

This will output:

```
DELETE FROM users WHERE name = 'dave' LIMIT 1
```

Similarly, PostgreSQL's *UPDATE* and *INSERT* query builders support a *RETURNING* clause:

```php
use RDev\Databases\SQL\QueryBuilders\PostgreSQL;

$updateQuery = (new PostgreSQL\QueryBuilder)->update("users", "", ["name" => "david"]);
    ->returning("id")
    ->addReturning("name");
echo $updateQuery->getSQL();
```

This will output:

```
UPDATE users SET name = ? RETURNING id, name
```

And

```php
use RDev\Databases\SQL\QueryBuilders\PostgreSQL;

$insertQuery = (new PostgreSQL\QueryBuilder)->insert("users", "", ["name" => "david"]);
    ->returning("id")
    ->addReturning("name");
echo $insertQuery->getSQL();
```

This will output:

```
INSERT INTO users (name) VALUES (?) RETURNING id, name
```