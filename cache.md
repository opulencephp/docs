# Cache

## Table of Contents
1. [Introduction](#introduction)
2. [Array Bridge](#array-bridge)
3. [File Bridge](#file-bridge)
4. [Memcached Bridge](#memcached-bridge)
5. [Redis Bridge](#redis-bridge)

<h2 id="introduction">Introduction</h2>
Cache is in-memory storage for frequently-accessed data.  Data in cache is commonly short-lived, but it can be persisted for long periods of time.  There are many caching libraries out there, such as Redis and Memcached, which offer tons of great features.  However, sometimes all your application needs is basic caching functions like:

1. `decrement($key, $by = 1)`
2. `delete($key)`
3. `flush()`
4. `get($key)`
5. `has($key)`
6. `increment($key, $by = 1)`
7. `set($key, $value, $lifetime)`

In this case, you can use a cache **bridge** to provide a simple wrapper around your favorite cache libraries.  RDev supplies the `RDev\Cache\ICacheBridge` interface with the above methods as well as some bridges to the most popular cache libraries.
  
<h2 id="array-bridge">Array Bridge</h2>
`RDev\Cache\ArrayBridge` provides a simple cache bridge most useful for running tests.

```php
use RDev\Cache\ArrayBridge;

$arrayBridge = new ArrayBridge();
```

<h2 id="file-bridge">File Bridge</h2>
`RDev\Cache\FileBridge` allows you to easily cache data to plaintext files on your server.

```php
use RDev\Cache\FileBridge;

$fileBridge = new FileBridge("/path/to/my/cache/files");
```

<h2 id="memcached-bridge">Memcached Bridge</h2>
`RDev\Cache\Memcached` acts as a simple wrapper around Memcached.  You can either use an instance of `Memcached` or `RDev\Memcached\RDevMemcached`.

```php
use RDev\Cache\MemcachedBridge;
use RDev\Memcached\RDevMemcached;
use RDev\Memcached\Server;
use RDev\Memcached\TypeMapper;

$memcached = new RDevMemcached(new TypeMapper());
$memcached->addServer(new Server("localhost", 11211));
$memcachedBridge = new MemcachedBridge($memcached);
```

You can add a prefix to all your keys to prevent naming collisions with other applications using your Memcached server:

```php
$memcachedBridge = new MemcachedBridge($memcached, "myapp:");
```

If you need the underlying Memcached instance to do anything beyond what the bridge does, you may call `getMemcached()`.

> **Note:** [Read more information](nosql#memcached) about RDev's Memcached extension.

<h2 id="redis-bridge">Redis Bridge</h2>
`RDev\Cache\Redis` is a simple bridge to Redis.  You can either use an instance of `Redis` or `RDev\Redis\RDevPHPRedis`.

```php
use RDev\Cache\RedisBridge;
use RDev\Redis\RDevPHPRedis;
use RDev\Redis\Server;
use RDev\Redis\TypeMapper;

$server = new Server("localhost");
$redis = new RDevPHPRedis($server, new TypeMapper());
$redisBridge = new RedisBridge($redis);
```

You can add a prefix to all your keys to prevent naming collisions with other applications using your Redis server:

```php
$redisBridge = new RedisBridge($redis, "myapp:");
```

If you need the underlying Redis instance to do anything beyond what the bridge does, you may call `getRedis()`.

> **Note:** [Read more information](nosql#redis) about RDev's Redis extension.