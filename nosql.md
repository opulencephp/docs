# NoSQL Databases

## Table of Contents
1. [Introduction](#introduction)
2. [Redis](#redis)
  1. [PHPRedis](#phpredis)
  2. [Predis](#predis)
3. [Memcached](#memcached)
  1. [Basic Memcached Usage](#basic-memcached-usage)

## Introduction
**RDev** provides developers with tools to read and write data from NoSQL (Not Only SQL) databases, which offer horizontal scalability, performance, and flexibility.  Often, NoSQL databases are used as a cache, giving applications enormous boosts in performance and scalability.  RDev provides extensions of popular NoSQL libraries such as Redis and Memcached to help your website scale.

## Redis
Redis is an extremely popular, in-memory key-value cache with pub/sub capabilities.  Unlike Memcached, Redis can store more complex structures such as sets, sorted lists, and hashes.  For more information, [please visit its homepage](http://redis.io/).

#### PHPRedis
**PHPRedis** is a Redis client extension to PHP written in C, giving you raw performance without the overhead of PHP scripts.  `RDevPHPRedis` extends PHPRedis and gives you the added feature of *type mappers* (provides methods for casting to and from Redis data types) and compatibility with `Server` objects.  To use one, simply:
```php
use RDev\Databases\NoSQL\Redis;

$phpRedis = new Redis\RDevPHPRedis(
    new Redis\Server(
        "localhost", // The host
        "mypassword", // The password, if there is one
        6379 // The port
    ),
    new Redis\TypeMapper()
);
$phpRedis->set("foo", "bar");
echo $phpRedis->get("foo"); // "bar"
```

#### Predis
**Predis** is a popular Redis client PHP library with the ability to create customized Redis commands.  `RDevPredis` extends Predis and gives you the added feature of *type mappers* and compatibility with `Server` objects.  To use one, simply:
```php
use RDev\Databases\NoSQL\Redis;

$predis = new Redis\RDevPredis(
    new Redis\Server(
        "localhost", // The host
        "mypassword", // The password, if there is one
        6379 // The port
    ),
    new Redis\TypeMapper()
);
$predis->set("foo", "bar");
echo $predis->get("foo"); // "bar"
```

## Memcached
Memcached (pronounced "Mem-cash-dee") is a distributed memory cache with basic key-value store functionality.  Although it doesn't come with all the bells and whistles of Redis, it does offer faster speed, which is suitable for simple key-value data.  For more information, [please visit its homepage](http://www.memcached.org/).

#### Basic Memcached Usage
```php
use RDev\Databases\NoSQL\Memcached;

$memcached = new Memcached\RDevMemcached(new Memcached\TypeMapper());
$memcached->addServer(new Memcached\Server(
    "localhost", // The host
    11211, // The port
    100 // The 'weight' of the server
));
$memcached->set("foo", "bar");
echo $memcached->get("foo"); // "bar"
```