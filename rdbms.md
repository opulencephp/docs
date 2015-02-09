# Relational Databases

## Table of Contents
1. [Introduction](#introduction)
3. [Single-Server Connection Pool](#single-server-connection-pool)
4. [Master-Slave Connection Pool](#master-slave-connection-pool)
5. [Read/Write Connections](#readwrite-connections)

<h2 id="introduction">Introduction</h2>
Relational databases store information about data and how it's related to other data.  **RDev** provides classes and methods for connecting to relational databases and querying them for data.  Connection pools help you manage your database connections by doing all the dirty work for you.  You can use an assortment of PHP drivers to connect to multiple types of server configurations.  For example, if you have a single database server in your stack, you can use a `SingleServerConnectionPool`.  If you have a master/slave(s) setup, you can use a `MasterSlaveConnectionPool`.
  
<h2 id="single-server-connection-pool">Single-Server Connection Pool</h2>
Single-server connection pools are useful for single-database server stacks, eg not master-slave setups.

```php
use RDev\Databases\SQL;
use RDev\Databases\SQL\PDO\MySQL;

$connectionPool = new SQL\SingleServerConnectionPool(
    new MySQL\Driver(), // The driver to use
    new SQL\Server(
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
use RDev\Databases\SQL;
use RDev\Databases\SQL\PDO\PostgreSQL;

$connectionPool = new SQL\SingleServerConnectionPool(
    new PostgreSQL\Driver(), // The driver to use
    new SQL\Server(
        "127.0.0.1", // The master host
        "username", // The master username
        "password", // The master password
        "databasename", // The name of the database to use
        3306 // The master port
    ),
    [
        // List any slave servers
        new SQL\Server(
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