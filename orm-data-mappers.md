# Data Mappers

## Table of Contents
1. [Introduction](#introduction)
2. [IDataMapper](#i-data-mapper)
  1. [Creating Custom get*() Methods](#creating-custom-get-methods)
3. [SQL Data Mappers](#sql-data-mappers)
  1. [Example](#sql-example)
4. [Cache Data Mappers](#cache-data-mappers)
  1. [Redis Data Mappers](#redis-data-mappers)
  2. [Example](#cache-example)
5. [Cached SQL Data Mappers](#cached-sql-data-mappers)
  1. [Example](#cached-sql-example)
6. [Generating Data Mapper Classes](#generating-data-mapper-classes)

<h2 id="introduction">Introduction</h2>
**Data mappers** act as the go-between for repositories and storage.  By abstracting this interaction away from repositories, you can swap your method of storage without affecting the repositories' interfaces.

<h2 id="i-data-mapper">IDataMapper</h2>
All data mappers must implement `RDev\ORM\DataMappers\IDataMapper`, which includes the following methods:
* `add()`
* `delete()`
* `getAll()`
* `getById()`
* `update()`

<h4 id="creating-custom-get-methods">Creating Custom get*() Methods</h4>
You'll frequently find yourself wanting to query entities by some criteria besides Id.  For example, you might want to look up posts by title using a `getByTitle()` method.  Let's create an interface with this method:

```php
namespace MyApp\Posts\ORM\DataMappers;
use RDev\ORM\DataMappers\IDataMapper;

interface IPostDataMapper extends IDataMapper
{
    public function getByTitle($title);
}
```

We'll implement this interface in the examples below.

<h2 id="sql-data-mappers">SQL Data Mappers</h2>
SQL data mappers use an SQL database for storage and querying.  They must implement the `RDev\ORM\DataMappers\ISQLDataMapper` (`SQLDataMapper` comes built-in).  An SQL data mapper implements all the methods from `IDataMapper` as well as `getIdGenerator()`, which returns the instance of the Id generator used by the data mapper.

> **Note:** Id generators are in the `RDev\ORM\Ids` namespace.

`SQLDataMapper` comes with a few extra methods built-in:

* `loadEntity()`
  * Accepts a row of data from an SQL query and converts it into an instance of the object maintained by the data mapper
  * You must implement this for each SQL data mapper
* `read()`
  * Executes the input query and parameters and loads the data into objects using the `loadEntity()` method
  * Useful for get*() methods
* `setIdGenerator()`
  * Sets the Id generator used by the data mapper
  * You must implement this for each SQL data mapper

<h4 id="sql-example">Example</h4>
Let's take a look at an example of an SQL data mapper for WordPress posts:

```php
namespace MyApp\WordPress\ORM\DataMappers;
use MyApp\WordPress\Post;
use PDO;
use RDev\Databases\IConnection;
use RDev\ORM\DataMappers\SQLDataMapper;
use RDev\ORM\Ids\IntSequenceIdGenerator;
use RDev\ORM\IEntity;

class PostSQLDataMapper extends SQLDataMapper implements IPostDataMapper
{
    /** @var Post $post */
    public function add(IEntity &$post)
    {
        $writeConnection = $this->connectionPool->getWriteConnection();
        $statement = $writeConnection->prepare(
            "INSERT INTO posts (content, title, author) VALUES (:content, :title, :author)"
        );
        $statement->bindValues([
            "content" => $post->getContent(),
            "title" => $post->getTitle(),
            "author" => $post->getAuthor()
        ]);
        $statement->execute();
    }
    
    /** @var Post $post */
    public function delete(IEntity &$post)
    {
        $writeConnection = $this->connectionPool->getWriteConnection();
        $statement = $writeConnection->prepare(
            "DELETE FROM posts WHERE id = :id"
        );
        $statement->bindValues([
            "id" => [$post->getId(), PDO::PARAM_INT]
        ]);
        $statement->execute();
    }
    
    public function getAll()
    {
        $sql = "SELECT id, content, title, author FROM posts";
        
        // The last parameter says that we are not expecting a single result
        return $this->read($sql, [], false);
    }
    
    public function getById($id)
    {
        $sql = "SELECT id, content, title, author FROM posts WHERE id = :id";
        $parameters = [
            "id" => [$id, PDO::PARAM_INT]
        ];
        
        // The last parameter says that we are expecting a single result
        return $this->read($sql, $parameters, true);
    }
    
    // This is a custom get*() method defined in IPostDataMapper
    public function getByTitle($title)
    {
        $sql = "SELECT id, content, title, author FROM posts WHERE title = :title";
        $parameters = [
            "title" => $title
        ];
        
        // The last parameter says that we are expecting a single result
        return $this->read($sql, $parameters, true);
    }
    
    /** @var Post $post */
    public function update(IEntity &$post)
    {
        $writeConnection = $this->connectionPool->getWriteConnection();
        $statement = $writeConnection->prepare(
            "UPDATE posts SET content = :content, title = :title, author = :author"
        );
        $statement->bindValues([
            "content" => $post->getContent(),
            "title" => $post->getTitle(),
            "author" => $post->getAuthor()
        ]);
        $statement->execute();
    }
    
    protected function loadEntity(array $hash, IConnection $connection)
    {
        // $hash contains the column values from a single row in the results
        return new Post(
            (int)$hash["id"],
            $hash["title"],
            $hash["author"]
        );
    }
    
    protected function setIdGenerator()
    {
        // This Id generator accepts the name of the Id sequence
        $this->idGenerator = new IntSequenceIdGenerator("posts_id_seq");
    }
}
```

> **Note:** The unit of work automatically handles setting the Id on the entity after the `add()` method is run as well as resetting it in case the transaction is rolled back.

<h2 id="cache-data-mappers">Cache Data Mappers</h2>
Cache data mappers use some form of cache (eg Redis or Memcached) for storage.  They must implement the `RDev\ORM\DataMappers\ICacheDataMapper` (`PHPRedisDataMapper` and `PredisDataMapper` come built-in).  A cache data mapper implements all the methods from `IDataMapper` as well as `flush()`, which flushes from cache all instances managed by the data mapper.

<h4 id="redis-data-mappers">Redis Data Mappers</h4>
Redis data mappers implement the following methods:

* `getEntityHashById()`
  * Gets a hash (row) of data from Redis for an entity
  * You must implement this for each Redis data mapper
* `getById()`
  * Gets an entity by an Id
* `loadEntity()`
  * Loads an entity from a hash of data
  * You must implement this for each Redis data mapper
* `read()`
  * Reads an entity or list of entities from an Id or list of Ids stored in a set or sorted set
  
<h4 id="cache-example">Example</h4>
Let's take a look at a PHPRedis data mapper example:

```php
namespace MyApp\WordPress\ORM\DataMappers;
use MyApp\WordPress\Post;
use RDev\ORM\DataMappers\PHPRedisDataMapper;
use RDev\ORM\IEntity;
use RDev\ORM\ORMException;

class PostRedisDataMapper extends PHPRedisDataMapper implements IPostDataMapper
{
    /** @var Post $post */
    public function add(IEntity &$post)
    {
        // Store a hash of the post's data
        if(!$this->redis->hMset("posts:" . $post->getID(), [
            "id" => $post->getID(),
            "title" => $post->getTitle(),
            "author" => $post->getAuthor()
        ]))
        {
            throw new ORMException("Failed to add post to Redis");
        }

        // Create an index of post Ids
        // This is useful for the getAll() method
        $this->redis->sAdd("posts", $post->getID());
        // Create an index of post titles
        // This is useful for the getByTitle() method
        $this->redis->set("posts:titles:{$post->getTitle()}", $post->getId());
    }
    
    /** @var Post $post */
    public function delete(IEntity &$post)
    {
        if(!$this->redis->del("posts:{$post->getID()}"))
        {
            throw new ORMException("Failed to delete post with ID {$post->getID()} from Redis");
        }

        // Remove this post from the index of posts
        $this->redis->sRemove("posts", $post->getID());
    }
    
    public function flush()
    {
        // Delete the index of posts as well as all posts
        if($this->redis->del("posts") === false || !$this->redis->deleteKeyPatterns("posts:*"))
        {
            throw new ORMException("Failed to flush posts from Redis");
        }
    }
    
    public function getAll()
    {
        // This will load all the Ids in the "posts" set and generate Post objects from them
        return $this->read("posts", self::VALUE_TYPE_SET);
    }
    
    // This is a custom get*() method defined in IPostDataMapper
    public function getByTitle($title)
    {
        // This will load the Id in the "posts:titles:$title" key and generate a Post object from it
        return $this->read("posts:titles:$title", self::VALUE_TYPE_STRING);
    }
    
    /** @var Post $post */
    public function update(IEntity &$post)
    {
        // In this case, an update will do the same thing as an addition
        $this->add($post);
    }
    
    protected function getEntityHashById($id)
    {
        $hash = $this->redis->hGetAll("posts:{$id}");

        if($hash == [])
        {
            // In the case that nothing was found, return null
            return null;
        }

        return $hash;
    }
    
    protected function loadEntity(array $hash)
    {
        return new Post(
            (int)$hash["id"],
            $hash["title"],
            $hash["author"]
        );
    }
}
```

<h2 id="cached-sql-data-mappers">Cached SQL Data Mappers</h2>
Cached SQL data mappers use an SQL database with a cache layer on top.  This reduces the number of queries going to your database by up to 95%, which drastically increases the speed of your data retrieval and improves scalability.  All writes are coordinated by the [unit of work](orm-units-of-work) so that cache and the SQL database are never out of sync.  Cached SQL data mappers contain a cache data mapper and an SQL data mapper.  It coordinates reads and writes between the two sub-data mappers and gives priority to the cache data mapper.  If data cannot be found in cache, it is queried from the SQL database and written back to cache for future queries.

> **Note:** The cache and SQL data mappers MUST implement the same interface for all `get*()` methods.  This allows the cached SQL data mapper to try and call a method from the cache data mapper, and if it fails, call it from the SQL data mapper.  If one of your data mappers does not return data for a particular method, just return `null` if the method returns a single entity, otherwise an empty array if it returns a list of entities.

Cached SQL data mappers must implement `RDev\ORM\DataMappers\ICachedSQLDataMapper` (`CachedSQLDataMapper` comes built-in).  `RedisCachedSQLDataMapper` and `MemcachedCachedSQLDataMapper` both extend `CachedSQLDataMapper` and provide functionality to simplify interactions with Redis and Memcached, respectively.

They come with the following methods built-in:

* `getUnsyncedEntities()`
  * Returns a list of entities in the following format:
    * "missing" => The list of entities that were not in cache
    * "differing" => The list of entities in cache that were not the same as in the SQL database
    * "additional" => The list of entities that appeared in cache, but not the SQL database
* `read()`
  * Reads from the sub-data mappers by giving cache the priority
  * Accepts the following parameters:
    1. `$funcName` - The name of the method to call in the sub-data mappers
    2. `$getFuncArgs` - The array of function arguments to pass in to our entity retrieval functions
    3. `$addDataToCacheOnMiss` - True if we want to add the entity from the database to cache in case of a cache miss
    4. `setFuncArgs` - The array of function arguments to pass into the set functions in the case of a cache miss
* `refreshEntities()`
  * Refreshes entities in cache that are not in sync with entities in the SQL database
  * Returns a list of the entities that were not in sync in the same format as `getUnsyncedEntities()`
* `setCacheDataMapper()`
  * Sets the instance of the sub-cache data mapper using the input cache instance
* `setSQLDataMapper()`
  * Sets the instance of the sub-SQL data mapper using the input connection pool instance
  
  
<h4 id="cached-sql-example">Example</h4>
Let's take a look at a cached SQL data mapper example that uses the cache and SQL data mappers from the previous examples:

```php
namespace MyApp\WordPress\ORM\DataMappers;
use RDev\Databases\ConnectionPool;
use RDev\ORM\DataMappers\RedisCachedSQLDataMapper;

class PostCachedSQLDataMapper extends RedisCachedSQLDataMapper implements IPostDataMapper
{
    public function getByTitle($title)
    {
        return $this->read("getByTitle", [$title]);
    }

    protected function setCacheDataMapper($cache)
    {
        $this->cacheDataMapper = new PostCacheDataMapper($cache);
    }

    protected function setSQLDataMapper(ConnectionPool $connectionPool)
    {
        $this->sqlDataMapper = new PostSQLDataMapper($connectionPool);
    }
}
```

Our `getByTitle()` method calls `$this->read()`, which automatically handles reading from cache and falling back to the SQL database on a cache miss.  This is all you need to do to take advantage of aggressive caching in your data mappers.

<h2 id="generating-data-mapper-classes">Generating Data Mapper Classes</h2>
You can use the console to generate any type of built-in data mapper using `php rdev make:datamapper`, and then selecting from the menu.