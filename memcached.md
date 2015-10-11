# Memcached

## Table of Contents
1. [Introduction](#introduction)
2. [Libraries](#libraries)
3. [Creating a Memcached Connection](#creating-memcached-connection)
  1. [Single Client](#single-client)
  2. [Multiple Clients](#multiple-clients)
  3. [Server Pools](#server-pools)
4. [Type Mappers](#type-mappers)
  1. [Booleans](#booleans)
  2. [Timestamps](#timestamps)

<h2 id="introduction">Introduction</h2>
Memcached (pronounced "Mem-cash-dee") is a distributed memory cache with basic key-value store functionality.  Although it doesn't come with all the bells and whistles of Redis, it does offer faster speed, which is suitable for simple key-value data.  For more information, <a href="http://php.net/manual/en/book.memcached.php" target="_blank">read the official PHP documentation</a>.

<h2 id="libraries">Libraries</h2>
Opulence lets you choose whichever Memcached library you'd like.  The built-in library `\Memcached` is the most popular.

<h2 id="creating-memcached-connection">Creating a Memcached Connection</h2>
`Opulence\Memcached\Memcached` accepts either a single client or a list of clients as well as a `TypeMapper` to help you convert to and from Memcached types.  Opulence uses magic methods to pass on method calls to the underlying Memcached client(s).

<h4 id="single-client">Single Client</h4>
```php
use Memcached as Client;
use Opulence\Memcached\Memcached;
use Opulence\Memcached\TypeMapper;

// Create our connection
$client = new Client();
$client->addServer("localhost", 11211);
$memcached = new Memcached($client, new TypeMapper());

// Try it out
$memcached->set("foo", "bar");
echo $memcached->get("foo"); // "bar"
```

You can get the client instance:

```php
$memcached->getClient();
```

<h4 id="multiple-clients">Multiple Clients</h4>
If you pass in multiple clients, one of them MUST be named `default`.

```php
use Memcached as Client;
use Opulence\Memcached\Memcached;
use Opulence\Memcached\TypeMapper;

$defaultClient = new Client();
$defaultClient->addServer("127.0.0.1", 11211);
$backupClient = new Client();
$backupClient->addServer("127.0.0.2", 11211);

$clients = [
    "default" => $defaultClient,
    "backup" => $backupClient
];
$memcached = new Memcached($clients, new TypeMapper());
```

You can get a particular client instance:

```php
$memcached->getClient("backup");
```

> **Note:** The `default` client will always be used unless you call `getClient($name)` and make calls to that client directly.

<h4 id="server-pools">Server Pools</h4>
`\Memcached` allows you to add multiple servers to a client:

```php
use Memcached as Client;
use Opulence\Memcached\Memcached;
use Opulence\Memcached\TypeMapper;

$client = new Client();
$client->addServer("127.0.0.1", 11211);
$client->addServer("127.0.0.2", 11211);
$memcached = new Memcached($client, new TypeMapper());
```

You can get the client instance:

```php
$memcached->getClient();
```

<h2 id="type-mappers">Type Mappers</h2>
`Opulence\Memcached\TypeMapper` helps you translate to and from Memcached data types.  For example, you cannot store a `DateTime` object in Memcached, so you need to convert to a Unix timestamp when storing it.  Conversely, when you read from Memcached, you can use a type mapper to convert the Unix timestamp back into a `DateTime` object.

You can get the type mapper object using `Opulence\Memcached\Memcached::getTypeMapper()`.

<h4 id="booleans">Booleans</h4>

##### To Memcached
```php
$phpBoolean = true;
echo $typeMapper->toMemcachedBoolean($phpBoolean); // 1
```

##### From Memcached
```php
$memcachedBoolean = 1;
echo $typeMapper->fromMemcachedBoolean($memcachedBoolean) === true; // 1
```

<h4 id="timestamps">Timestamps</h4>

##### To Memcached
```php
$phpDate = new DateTime("1987-07-24 12:34:56");
echo $typeMapper->toMemcachedTimestamp($phpDate); // 554128496
```

##### From Memcached
```php
$memcachedDate = 554128496;
$phpDate = $typeMapper->fromMemcachedTimestamp($memcachedDate);
echo $phpDate->format("Y-m-d"); // "1987-07-24"
```