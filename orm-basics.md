# Object-Relational Mapping

## Table of Contents
1. [Introduction](#introduction)
2. [Repositories](#repositories)
3. [Data Mappers](#data-mappers)
4. [Automatic Caching](#automatic-caching)

<h2 id="introduction">Introduction</h2>
**RDev** utilizes the *repository pattern* to encapsulate data retrieval from storage.  *Repositories* have *data mappers* which actually interact directly with storage, eg cache and/or a relational database.  Repositories use [*units of work*](orm-units-of-work), which act as transactions across multiple repositories.  Unlike other popular PHP frameworks, RDev does not force you to extend ORM classes in order to make them storable.  The only interface they must implement is `RDev\ORM\IEntity`, which simply requires `getId()` and `setId()`.

<h2 id="repositories">Repositories</h2>
*Repositories* act as collections of entities.  They include methods for adding, deleting, and retrieving entities.  The actual retrieval from storage is done through *data mappers* contained in the repository.  Note that there are no methods like `update()` or `save()`.  These actions take place in the *data mapper* and are scheduled by the *unit of work* contained by the repository.  [Read here](#data-mappers) for more information on DataMappers or [here](#unit-of-work) for more information on units of work.

> **Note:** In `get*()` repository methods, do not call the data mapper directly.  Instead, call `getFromDataMapper()`, which will handle managing entities in the unit of work.

<h2 id="data-mappers">Data Mappers</h2>
*Data mappers* act as the go-between for repositories and storage.  By abstracting this interaction away from repositories, you can swap your method of storage without affecting the repositories' interfaces.  There are currently 3 types of DataMappers, but you can certainly add your own by implementing `RDev\ORM\DataMappers\IDataMapper`:

1. `SQLDataMapper`
  * Uses an SQL database as its method of storage
  * Allows you to write your own SQL queries to read and write to the database
2. `RedisDataMapper`
  * Uses Redis as its method of storage
    * Can use PHPRedis or Predis as the Redis client library
  * Allows you to write your own methods to read and write data to Redis
3. `CachedSQLDataMapper`
  * Uses an *SQLDataMapper* as its primary storage and an `ICacheDataMapper` to read and write from cache
  * Drastically reduces the number of SQL queries and improves performance through heavy caching
  * `RedisCachedSQLDataMapper` and `MemcachedCachedSQLDataMapper` extend `CachedSQLDataMapper` to give you Redis- and Memcached-backed data mappers, respectively
  * Can get a list of entities that are not in sync between cache and the SQL database using `getUnsyncedEntities()`
    * Returns an array with the following keys:
      * "missing" => The list of entities that were not in cache
      * "differing" => The list of entities in cache that were not the same as in the SQL database
      * "additional" => The list of entities that appeared in cache, but not the SQL database
  * Can synchronize entities in cache with those in the SQL database using `refreshEntities()`
    * Returns an array with the same format as `getUnsyncedEntities()`

```

<h2 id="automatic-caching">Automatic Caching</h2>
By extending the `CachedSQLDataMapper`, you can take advantage of automatic caching of entities for faster performance.  Entities are searched in cache before defaulting to an SQL database, and they are added to cache on misses.  Writes to cache are automatically queued whenever writing to a `CachedSQLDataMapper`.  To keep cache in sync with SQL, the writes are only performed once a unit of work commits successfully.