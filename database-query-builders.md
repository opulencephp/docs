# Query Builders

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Usage](#basic-usage)
3. [Clauses](#clauses)
  1. [Conditions](#conditions)
4. [Binding Values](#binding-values)
5. [Select Queries](#select-queries)
6. [Insert Queries](#insert-queries)
7. [Update Queries](#update-queries)
8. [Delete Queries](#delete-queries)
9. [Using Query Builders with PDO](#using-query-builders-with-pdo)
10. [Vendor-Specific Query Builders](#vendor-specific-query-builders)

<h2 id="introduction">Introduction</h2>
Sometimes you need to programmatically generate SQL queries.  Rather than concatenating strings together, you can use `QueryBuilders` to do the heavy lifting.  They provide a fluent syntax for creating queries and binding values to queries.  This library is adapter-agnostic, meaning you can use it with any database adapter or with any library, such as PHP's `PDOStatement` or [Opulence's PDO wrapper](database-basics).  Query builders even support vendor-specific query features, such as MySQL's `LIMIT` clause support for `DELETE` statements.

<h2 id="basic-usage">Basic Usage</h2>
Let's look at a simple `SELECT` query:

```php
use Opulence\QueryBuilders\PostgreSql\QueryBuilder;

$query = (new QueryBuilder)->select('id', 'name', 'email')
    ->from('users')
    ->where('datejoined < NOW()');
echo $query->getSql();
```

This will output:
```
SELECT id, name, email FROM users WHERE datejoined < NOW()
```

<h2 id="clauses">Clauses</h2>
`QueryBuilders` support a variety of clauses.  You may use the following clauses to build complex, but easy-to-read-and-maintain queries:

* *FROM*
  * `from($tableName, $tableAlias)`
* *WHERE*
  * `where($condition)`
  * `andWhere($condition)`
  * `orWhere($condition)`
* *JOIN*
  * `join($tableName, $tableAlias, $condition)`
  * `innerJoin($tableName, $tableAlias, $condition)`
  * `leftJoin($tableName, $tableAlias, $condition)`
  * `rightJoin($tableName, $tableAlias, $condition)`
* *GROUP*
  * `groupBy($expression)`
  * `addGroupBy($expression)`
* *HAVING*
  * `having($condition)`
  * `andHaving($condition)`
  * `orHaving($condition)`
* *ORDER BY*
  * `orderBy($condition)`
  * `addOrderBy($expression)`
* *LIMIT*
  * `limit($numRows)`
  * `offset($numRows)`
* *RETURNING* (PostgreSQL only)
  * `returning($expression)`
  * `addReturning($expression)`

<h3 id="conditions">Conditions</h3>
Opulence provides an easy way to add conditions to your queries using `Opulence\QueryBuilders\Conditions\ConditionFactory`.  It provides methods for creating the following conditions in your `where()` and `having()` clauses:

* `between(string $column, $min, $max, int $dataType = \PDO::PARAM_STR)`
* `in(string $column, array $parameters)`
  * Parameter values can either be the value itself or an array whose first item is the value and whose second value is the PDO data type, eg `\PDO::PARAM_INT`
* `in(string $column, array $subExpressions)`
  * Sub-expressions can be any valid SQL expressions such as sub-queries

You can also negate these methods:

* `notBetween(...)`
* `notIn(...)`

Here's an example of how to grab all users whose Id matches at least one of the input values:

```php
use Opulence\QueryBuilders\Conditions\ConditionFactory;

$conditions = new ConditionFactory();
$query = (new QueryBuilder)->select('name')
    ->from('users')
    ->where($conditions->in('id', [[23, \PDO::PARAM_INT], [33, \PDO::PARAM_INT]]));
```

Using `ConditionFactory` will automatically bind any values as [unnamed placeholders](#binding-values).

> **Note:** If you're trying to include complex expressions in your conditions, eg `birthday BETWEEN NOW() AND CAST('2050-01-01' AS Date)`, you're best off just writing them as strings in the `where()` or `having()` clauses.

<h2 id="binding-values">Binding Values</h2>
`QueryBuilders` provide an intuitive syntax for binding values to queries ([learn more about statement bindings](database-basics#binding-values)).  To add a named placeholder, use `addNamedPlaceholderValue()`:

```php
$query = (new QueryBuilder)->select('content')
    ->from('posts')
    ->where('id < :id')
    ->addNamedPlaceholderValue('id', 24, \PDO::PARAM_INT);
```

To add many named placeholder values, use `addNamedPlaceholderValues()`:

```php
$query = (new QueryBuilder)->select('count(*)')
    ->from('users')
    ->where('username = :username')
    ->orWhere('id = :id')
    ->addNamedPlaceholderValues([
        // Non-array values are assumed to be of type \PDO::PARAM_STR
        'username' => 'dave_y',
        // In array values, the first item is the value, and the second is the parameter type
        'id'       => [24, \PDO::PARAM_INT]
    ]);
```

Similarly, `addUnnamedPlaceholderValue()` and `addUnnamedPlaceholderValues()` can be used to add unnamed placeholder values.

> **Note:** You cannot mix named with unnamed placeholders.  Also, if no type is specified for a bound value, it's assumed to be \PDO::PARAM_STR.

<h2 id="select-queries">Select Queries</h2>
Select queries use a variable argument list to specify the columns to select:

```php
$query = (new QueryBuilder)->select('title', 'author')
    ->from('books');
echo $query->getSql();
```

This will output:

```
SELECT title, author FROM books
```

<h2 id="insert-queries">Insert Queries</h2>
Insert queries accept a table name and a mapping of column names to values:

```php
$query = (new QueryBuilder)->insert('users', [
    'name'  => 'Brian',
    'email' => 'foo@bar.com',
    'age'   => [24, \PDO::PARAM_INT]
]);
echo $query->getSql();
```

This will output:

```
INSERT INTO users (name, email, age) VALUES (?, ?, ?)
```

The following values are bound to the query:

```php
[
    ['Brian', \PDO::PARAM_STR],
    ['foo@bar.com', \PDO::PARAM_STR],
    [24, \PDO::PARAM_INT]
]
```

> **Note:** `INSERT` and `UPDATE` query builders bind unnamed placeholder values.  To specify the type of the value, use an array whose first item is the value and whose second item is the type.

<h2 id="update-queries">Update Queries</h2>
Update queries accept a table name, table alias, and a mapping of column names to values:

```php
$query = (new QueryBuilder)->update('users', 'u', [
    'name' => 'Dave',
    'age'  => [24, \PDO::PARAM_INT]
    ])
    ->where('id = ?')
    ->addUnnamedPlaceholderValue(1234, \PDO::PARAM_INT);
echo $query->getSql();
```

This will output:

```
UPDATE users AS u SET name = ?, age = ? WHERE id = ?
```

The following values are bound to the query:

```php
[
    ['Dave', \PDO::PARAM_STR],
    [24, \PDO::PARAM_INT],
    [1234, \PDO::PARAM_INT]
]
```

> **Note:** Like `INSERT` query builders, `UPDATE` query builders bind unnamed placeholder values.

<h2 id="delete-queries">Delete Queries</h2>
Delete queries accept a table name:

```php
$query = (new QueryBuilder)->delete('users')
    ->where('id = :id');
echo $query->getSql();
```

This will output:

```
DELETE FROM users WHERE id = :id
```

<h2 id="using-query-builders-with-pdo">Using Query Builders with PDO</h2>
Let's say you've built the following query:

```php
$query = (new QueryBuilder)->select('author')
    ->from('books')
    ->where('title = :title')
    ->addNamedPlaceholderValue('title', 'Code Complete');
```

Simply call `getSql()` and `getParameters()` to use this in `PDO` or in [Opulence's PDO wrapper](database-basics):

```php
$statement = $connection->prepare($query->getSql());
$statement->bindValues($query->getParameters());
$statement->execute();
```

<h2 id="vendor-specific-query-builders">Vendor-Specific Query Builders</h2>
MySQL and PostgreSQL have their own query builders, which implement features that are unique to each database.  For example, the MySQL query builder supports a *LIMIT* clause:

```php
use Opulence\QueryBuilders\MySql\QueryBuilder;

$query = (new QueryBuilder)->delete('users')
    ->where("name = 'Dave'")
    ->limit(1);
echo $query->getSql();
```

This will output:

```
DELETE FROM users WHERE name = 'Dave' LIMIT 1
```

Similarly, PostgreSQL's `UPDATE` and `INSERT` query builders support a *RETURNING* clause:

```php
use Opulence\QueryBuilders\PostgreSql\QueryBuilder;

$query = (new QueryBuilder)->update('users', '', [
    'status' => [0, \PDO::PARAM_INT]
    ])
    ->returning('id')
    ->addReturning('name');
echo $query->getSql();
```

This will output:

```
UPDATE users SET status = ? RETURNING id, name
```

The following values are bound to the query:

```php
[
    [0, \PDO::PARAM_INT]
]
```

Here's an example of an `INSERT` statement with a *RETURNING* clause:

```php
use Opulence\QueryBuilders\PostgreSql\QueryBuilder;

$query = (new QueryBuilder)->insert('users', '', ['name' => 'David'])
    ->returning('id')
    ->addReturning('name');
echo $query->getSql();
```

This will output:

```
INSERT INTO users (name) VALUES (?) RETURNING id, name
```

The following values are bound to the query:

```php
[
    ['David', \PDO::PARAM_STR]
]
```
