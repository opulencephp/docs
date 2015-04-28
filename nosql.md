# NoSQL Databases

## Table of Contents
1. [Introduction](#introduction)
2. [Redis](#redis)
  1. [PHPRedis](#phpredis)
  2. [Predis](#predis)
3. [Memcached](#memcached)
  1. [Basic Memcached Usage](#basic-memcached-usage)

<h2 id="introduction">Introduction</h2>
**RDev** provides developers with tools to read and write data from NoSQL (Not Only SQL) databases, which offer horizontal scalability, performance, and flexibility.  Often, NoSQL databases are used as a cache, giving applications enormous boosts in performance and scalability.  RDev provides extensions of popular NoSQL libraries such as Redis and Memcached to help your website scale.

<h2 id="redis">Redis</h2>
Redis is an extremely popular, in-memory key-value cache with pub/sub capabilities.  Unlike Memcached, Redis can store more complex structures such as sets, sorted lists, and hashes.  For more information, <a href="http://redis.io/" target="_blank">please visit its homepage</a>.

<h4 id="phpredis">PHPRedis</h4>
<a href="https://github.com/phpredis/phpredis" target="_blank">PHPRedis</a> is a Redis client extension to PHP written in C, giving you raw performance without the overhead of PHP scripts.  `RDevPHPRedis` extends PHPRedis and gives you the added feature of *type mappers* (provides methods for casting to and from Redis data types) and compatibility with `Server` objects.  To use one, simply:
```php
use RDev\Redis\RDevPHPRedis;
use RDev\Redis\Server;
use RDev\Redis\TypeMapper;

$phpRedis = new RDevPHPRedis(
    new Server(
        "localhost", // The host
        "mypassword", // The password, if there is one
        6379 // The port
    ),
    new TypeMapper()
);
$phpRedis->set("foo", "bar");
echo $phpRedis->get("foo"); // "bar"
```

<h4 id="predis">Predis</h4>
<a href="https://github.com/nrk/predis" target="_blank">Predis</a> is a popular Redis client PHP library with the ability to create customized Redis commands.  `RDevPredis` extends Predis and gives you the added feature of *type mappers* and compatibility with `Server` objects.  To use one, simply:
```php
use RDev\Redis\RDevPredis;
use RDev\Redis\Server;
use RDev\Redis\TypeMapper;

$predis = new RDevPredis(
    new Server(
        "localhost", // The host
        "mypassword", // The password, if there is one
        6379 // The port
    ),
    new TypeMapper()
);
$predis->set("foo", "bar");
echo $predis->get("foo"); // "bar"
```

<h2 id="memcached">Memcached</h2>
Memcached (pronounced "Mem-cash-dee") is a distributed memory cache with basic key-value store functionality.  Although it doesn't come with all the bells and whistles of Redis, it does offer faster speed, which is suitable for simple key-value data.  For more information, <a href="http://php.net/manual/en/book.memcached.php" target="_blank">read the official PHP documentation</a>.

<h4 id="basic-memcached-usage">Basic Memcached Usage</h4>
```php
use RDev\Memcached\RDevMemcached;
use RDev\Memcached\Server;
use RDev\Memcached\TypeMapper;

$memcached = new RDevMemcached(new TypeMapper());
$memcached->addServer(new Server(
    "localhost", // The host
    11211, // The port
    100 // The 'weight' of the server
));
$memcached->set("foo", "bar");
echo $memcached->get("foo"); // "bar"
```