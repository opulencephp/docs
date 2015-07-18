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

In this case, you can use a cache **bridge** to provide a simple wrapper around your favorite cache libraries.  Opulence supplies the `Opulence\Cache\ICacheBridge` interface with the above methods as well as some bridges to the most popular cache libraries.
  
<h2 id="array-bridge">Array Bridge</h2>
`Opulence\Cache\ArrayBridge` provides a simple cache bridge most useful for running tests.

```php
use Opulence\Cache\ArrayBridge;

$arrayBridge = new ArrayBridge();
```

<h2 id="file-bridge">File Bridge</h2>
`Opulence\Cache\FileBridge` allows you to easily cache data to plaintext files on your server.

```php
use Opulence\Cache\FileBridge;

$fileBridge = new FileBridge("/path/to/my/cache/files");
```

<h2 id="memcached-bridge">Memcached Bridge</h2>
`Opulence\Cache\Memcached` acts as a simple wrapper around Memcached.  You can either use an instance of `Memcached` or `Opulence\Memcached\OpulenceMemcached`.

```php
use Opulence\Cache\MemcachedBridge;
use Opulence\Memcached\OpulenceMemcached;
use Opulence\Memcached\Server;
use Opulence\Memcached\TypeMapper;

$memcached = new OpulenceMemcached(new TypeMapper());
$memcached->addServer(new Server("localhost", 11211));
$memcachedBridge = new MemcachedBridge($memcached);
```

You can add a prefix to all your keys to prevent naming collisions with other applications using your Memcached server:

```php
$memcachedBridge = new MemcachedBridge($memcached, "myapp:");
```

If you need the underlying Memcached instance to do anything beyond what the bridge does, you may call `getMemcached()`.

> **Note:** [Read more information](nosql#memcached) about Opulence's Memcached extension.

<h2 id="redis-bridge">Redis Bridge</h2>
`Opulence\Cache\Redis` is a simple bridge to Redis.  You can either use an instance of `Redis` or `Opulence\Redis\OpulencePHPRedis`.

```php
use Opulence\Cache\RedisBridge;
use Opulence\Redis\OpulencePHPRedis;
use Opulence\Redis\Server;
use Opulence\Redis\TypeMapper;

$server = new Server("localhost");
$redis = new OpulencePHPRedis($server, new TypeMapper());
$redisBridge = new RedisBridge($redis);
```

You can add a prefix to all your keys to prevent naming collisions with other applications using your Redis server:

```php
$redisBridge = new RedisBridge($redis, "myapp:");
```

If you need the underlying Redis instance to do anything beyond what the bridge does, you may call `getRedis()`.

> **Note:** [Read more information](nosql#redis) about Opulence's Redis extension.