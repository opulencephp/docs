# Database Basics

## Table of Contents
1. [Introduction](#introduction)
  1. [Connection Pools](#connection-pools)
2. [Single-Server Connection Pool](#single-server-connection-pool)
3. [Master-Slave Connection Pool](#master-slave-connection-pool)
4. [Read/Write Connections](#readwrite-connections)
5. [How to Query and Fetch Results](#how-to-query-and-fetch-results)
6. [Binding Values](#binding-values)
  1. [Binding Named Placeholders](#binding-named-placeholders)
  2. [Binding Unnamed Placeholders](#binding-unnamed-placeholders)
  3. [Binding Multiple Values](#binding-multiple-values)

<h2 id="introduction">Introduction</h2>
Relational databases store information about data and how it's related to other data.  RDev provides classes and methods for connecting to relational databases and querying them for data.  It does this by extending `PDO` and `PDOStatement` to give users a familiar interface to work with.  <a href="http://php.net/manual/en/book.pdo.php" target="_blank">PDO</a> is a powerful wrapper for database interactions, and comes with built-in tools to prevent SQL injection. 

<h4 id="connection-pools">Connection Pools</h4>
Connection pools help you manage your database connections by doing all the dirty work for you.  You can use an assortment of PHP drivers to connect to multiple types of server configurations.  For example, if you have a single database server in your stack, you can use a `SingleServerConnectionPool`.  If you have a master/slave(s) setup, you can use a `MasterSlaveConnectionPool`.
  
<h2 id="single-server-connection-pool">Single-Server Connection Pool</h2>
Single-server connection pools are useful for single-database server stacks, eg not master-slave setups.

```php
use RDev\Databases\PDO\MySQL\Driver;
use RDev\Databases\Server;
use RDev\Databases\SingleServerConnectionPool;

$connectionPool = new SingleServerConnectionPool(
    new Driver(), // The driver to use
    new Server(
        "localhost", // The host
        "username", // The server username
        "password", // The server password
        "databasename", // The name of the database to use
        3306 // The port
    ),
    [], // Any settings that help setup a driver connection, eg "unix_socket" for MySQL Unix sockets
    [] // Any driver-specific connection settings, eg \PDO::ATTR_PERSISTENT => true
);
$readConnection = $connectionPool->getReadConnection();
// The next part should be familiar to people that have used PDO
$statement = $readConnection->prepare("SELECT name FROM users WHERE id = :id");
$statement->bindValue("id", 1234, \PDO::PARAM_INT);
$statement->execute();
$row = $statement->fetch(\PDO::FETCH_ASSOC);
// This will contain the user's name whose Id is 1234
$name = $row["name"];
```

<h2 id="master-slave-connection-pool">Master-Slave Connection Pool</h2>
Master-slave connection pools are useful for setups that include a master and at least one slave server.  Instead of taking a single server in their constructors, they take a master server and an array of slave servers.

```php
use RDev\Databases\PDO\PostgreSQL\Driver;
use RDev\Databases\Server;
use RDev\Databases\SingleServerConnectionPool;

$connectionPool = new SingleServerConnectionPool(
    new Driver(), // The driver to use
    new Server(
        "127.0.0.1", // The master host
        "username", // The master username
        "password", // The master password
        "databasename", // The name of the database to use
        3306 // The master port
    ),
    [
        // List any slave servers
        new Server(
            "127.0.0.2", // The slave host
            "username", // The slave username
            "password", // The slave password
            "databasename", // The name of the database to use
            3306 // The slave port
        )
    ],
    [], // Any settings that help setup a driver connection, eg "unix_socket" for MySQL Unix sockets
    [] // Any driver-specific connection settings, eg \PDO::ATTR_PERSISTENT => true
);
```

<h2 id="readwrite-connections">Read/Write Connections</h2>
To read from the database, simply use the connection returned by `$connectionPool->getReadConnection()`.  Similarly, `$connectionPool->getWriteConnection()` will return a connection to use for write queries.  These two methods take care of figuring out which server to connect to.  If you want to specify a server to connect to, you can pass it in as a parameter to either of these methods.

<h2 id="how-to-query-and-fetch-results">How to Query and Fetch Results</h2>
RDev uses the exact same methods as `PDO` to query and fetch results.  To learn how to query using `PDO`, try the <a href="http://php.net/manual/en/book.pdo.php" target="_blank">official PHP documentation</a>.

RDev's `PDO` wrappers make it easy to connect to the database without having to remember things like how to format the DSN.  RDev's wrappers also support [type mappers](database-type-mappers) for easy conversion between a database vendor's data types and PHP data types.  They even provide support for nested database transactions.

<h2 id="binding-values">Binding Values</h2>
Most database queries use a dynamic variable to filter results.  The unsafe method would be to put it directly in the string:

```php
$id = 24;
$query = "SELECT email FROM users WHERE id = $id";
```

The issue here is what's called *SQL injection*.  What would happen if a malicious user input "1 OR 1=1" into the query above?  We'd get:

```php
"SELECT email FROM users WHERE id = 1 OR 1=1"
```

See the issue there?  The malicious user just tricked your application into returning the email address for every user.  This is where *prepared statements* and binding comes in handy.  Instead of just concatenating your value into the query, `PDO` will automatically escape the data before using it in the query.

> **Note:** For data binding to work properly, it is imperative that you include the type of the parameter being bound, eg `PDO::PARAM_INT` or `PDO::PARAM_BOOL`.

<h4 id="binding-named-placeholders">Binding Named Placeholders</h4>
It's convenient to name placeholders that you'll bind to in a query so that you can reference them by name:

```php
$id = 24;
$statement = $connection->prepare("SELECT title FROM posts WHERE id = :id");
$statement->bindValue("id", $id, \PDO::PARAM_INT);
$statement->execute();
```

<h4 id="binding-unnamed-placeholders">Binding Unnamed Placeholders</h4>
It's also possible to bind to unnamed placeholders in the case that the number of parameters you're binding is dynamic:

```php
$statement = $connection->prepare("SELECT title FROM posts WHERE id = ?");
// Unnamed placeholders are 1-indexed
$statement->bindValue(1, $id, \PDO::PARAM_INT);
$statement->execute();
```

<h4 id="binding-multiple-values">Binding Multiple Values</h4>
`PDOStatement` has a `bindValue()` method, but it does not natively support binding multiple values at once.  RDev's extension of `PDOStatement` does:

```php
$statement->bindValues([
    // By default, values are interpreted as type PDO::PARAM_STR
    "name" => "Dave",
    // To bind a non-string type to a value, use an array
    // The first item is the value, and the second is the parameter type
    "id" => [23, \PDO::PARAM_INT]
]);
```

You can also bind unnamed placeholder values:

```php
$statement->bindValues([
    "Boeing",
    [727, \PDO::PARAM_INT]
]);
```