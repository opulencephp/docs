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

$fileBridge = new FileBridge('/path/to/my/cache/files');
```

<h2 id="memcached-bridge">Memcached Bridge</h2>

`Opulence\Cache\MemcachedBridge` acts as a simple wrapper around Memcached.

```php
use Memcached as Client;
use Opulence\Cache\MemcachedBridge;
use Opulence\Memcached\Memcached;

$client = new Client();
$client->addServer('localhost', 11211);
$memcached = new Memcached($client);
$memcachedBridge = new MemcachedBridge($memcached);
```

If you would like to use a Memcached client besides the "default" one, specify it:

```php
$memcachedBridge = new MemcachedBridge($memcached, 'some-other-server');
```

You can add a prefix to all your keys to prevent naming collisions with other applications using your Memcached server:

```php
$memcachedBridge = new MemcachedBridge($memcached, 'default', 'myapp:');
```

If you need the underlying Memcached instance to do anything beyond what the bridge does, you may call `getMemcached()`.

> **Note:** [Read more information](memcached) about Opulence's Memcached extension.

<h2 id="redis-bridge">Redis Bridge</h2>

`Opulence\Cache\RedisBridge` is a simple bridge to Redis.

```php
use Opulence\Cache\RedisBridge;
use Opulence\Redis\Redis;
use Redis as Client;

$client = new Client();
$client->connect('localhost', 6379);
$redis = new Redis($client);
$redisBridge = new RedisBridge($redis);
```

If you would like to use a Redis client besides the "default" one, specify it:

```php
$redisBridge = new RedisBridge($redis, 'some-other-server');
```

You can add a prefix to all your keys to prevent naming collisions with other applications using your Redis server:

```php
$redisBridge = new RedisBridge($redis, 'default', 'myapp:');
```

If you need the underlying Redis instance to do anything beyond what the bridge does, you may call `getRedis()`.

> **Note:** [Read more information](redis) about Opulence's Redis extension.
