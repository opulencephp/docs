# Redis

## Table of Contents
1. [Introduction](#introduction)
2. [Libraries](#libraries)
3. [Creating a Redis Connection](#creating-redis-connection)
  1. [Single Client](#single-client)
  2. [Multiple Clients](#multiple-clients)
  3. [Clustering](#clustering)
4. [Type Mappers](#type-mappers)
  1. [Booleans](#booleans)
  2. [Timestamps](#timestamps)

<h2 id="introduction">Introduction</h2>
Redis is an extremely popular, in-memory key-value cache with pub/sub capabilities.  Unlike Memcached, Redis can store more complex structures such as sets, sorted lists, and hashes.  For more information, <a href="http://redis.io/" target="_blank">please visit its homepage</a>.

<h2 id="libraries">Libraries</h2>
Opulence lets you choose whichever Redis library you'd like.  The following are the two most popular:

1. <a href="https://github.com/phpredis/phpredis" target="_blank">PHPRedis</a>
  * PHP extension written in C, giving you raw performance
2. <a href="https://github.com/nrk/predis" target="_blank">Predis</a>
  * PHP library that does not require you to re-compile PHP

<h2 id="creating-redis-connection">Creating a Redis Connection</h2>
`Opulence\Redis\Redis` acts as a convenient wrapper around Redis.  It accepts either a single client or a list of clients.  Opulence uses magic methods to pass on method calls to the underlying Redis client(s).

<h4 id="single-client">Single Client</h4>
```php
use Opulence\Redis\Redis;
use Redis as Client;

// Create our connection
$client = new Client();
$client->connect("localhost", 6379);
$redis = new Redis($client);

// Try it out
$redis->set("foo", "bar");
echo $redis->get("foo"); // "bar"
```

You can get the client instance:

```php
$redis->getClient();
```

<h4 id="multiple-clients">Multiple Clients</h4>
If you pass in multiple clients, one of them MUST be named `default`.

```php
use Opulence\Redis\Redis;
use Redis as Client;

$defaultClient = new Client();
$defaultClient->connect("127.0.0.1", 6379);
$backupClient = new Client();
$backupClient->connect("127.0.0.2", 6379);
$clients = [
    "default" => $defaultClient,
    "backup" => $backupClient
];
$redis = new Redis($clients);
```

You can get a particular client instance:

```php
$redis->getClient("backup");
```

> **Note:** The `default` client will always be used unless you call `getClient($name)` and make calls to that client directly.

<h4 id="clustering">Clustering</h4>
Redis 3.0 added the ability to automatically shard your Redis database across a cluster.  Let's take a look at how we can use Predis to connect to a cluster:

```php
use Opulence\Redis\Redis;
use Predis\Client;

$client = new Client(
    [
        "tcp://127.0.0.1",
        "tcp://127.0.0.2"
    ],
    [
        "cluster" => "redis"
    ]
);
$redis = new Redis($client);
```

You can get the client instance:

```php
$redis->getClient();
```

<h2 id="type-mappers">Type Mappers</h2>
`Opulence\Redis\Types\TypeMapper` helps you translate to and from Redis data types.  For example, you cannot store a `DateTime` object in Redis, so you need to convert to a Unix timestamp when storing it.  Conversely, when you read from Redis, you can use a type mapper to convert the Unix timestamp back into a `DateTime` object.

You can also use a factory to create type mappers:

```php
use Opulence\Redis\Types\Factories\TypeMapperFactory;

$typeMapper = (new TypeMapperFactory)->createTypeMapper();
```

<h4 id="booleans">Booleans</h4>

##### To Redis
```php
$phpBoolean = true;
echo $typeMapper->toRedisBoolean($phpBoolean); // 1
```

##### From Redis
```php
$redisBoolean = 1;
echo $typeMapper->fromRedisBoolean($redisBoolean) === true; // 1
```

<h4 id="timestamps">Timestamps</h4>

##### To Redis
```php
$phpDate = new DateTime("1987-07-24 12:34:56");
echo $typeMapper->toRedisTimestamp($phpDate); // 554128496
```

> **Note:** This method accepts any object implementing `DateTimeInterface`, including `DateTimeImmutable`.

##### From Redis
```php
$redisDate = 554128496;
$phpDate = $typeMapper->fromRedisTimestamp($redisDate);
echo $phpDate->format("Y-m-d"); // "1987-07-24"
```