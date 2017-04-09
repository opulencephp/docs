# Data Mappers

## Table of Contents
1. [Introduction](#introduction)
2. [IDataMapper](#i-data-mapper)
  1. [Creating Custom get*() Methods](#creating-custom-get-methods)
3. [SQL Data Mappers](#sql-data-mappers)
  1. [Example](#sql-example)
  2. [Query Builders](#query-builders)
4. [Cache Data Mappers](#cache-data-mappers)
  1. [Redis Data Mappers](#redis-data-mappers)
  2. [Example](#cache-example)
5. [Cached SQL Data Mappers](#cached-sql-data-mappers)
  1. [Example](#cached-sql-example)
6. [Relationships](#relationships)
7. [Generating Data Mapper Classes](#generating-data-mapper-classes)

<h2 id="introduction">Introduction</h2>
**Data mappers** act as the go-between for repositories and storage.  By abstracting this interaction away from repositories, you can abstract your storage mechanism away from your business logic.  Opulence's ORM does not force your domain models to implement some interface or extend some base class.  Instead, you can use plain-old PHP objects (POPOs), keeping the framework out of your business logic.

For reference our examples below, lets assume our `Post` class looks something like this:

```php
namespace MyApp\Posts;

class Post
{
    private $id;
    private $title;
    private $text;
    private $author;

    public function __construct(int $id, string $title, string $author, string $text)
    {
        $this->id = $id;
        $this->title = $title;
        $this->author = $author;
        $this->text = $text;
    }

    public function getAuthor() : string
    {
        return $this->author;
    }

    public function getId() : int
    {
        return $this->id;
    }

    public function getText() : string
    {
        return $this->text;
    }

    public function getTitle() : string
    {
        return $this->title;
    }

    public function setId(int $id) : void
    {
        $this->id = $id;
    }
}
```

<h2 id="i-data-mapper">IDataMapper</h2>
All data mappers must implement `Opulence\Orm\DataMappers\IDataMapper`, which includes the following methods:
* `add()`
* `delete()`
* `getAll()`
* `getById()`
* `update()`

<h4 id="creating-custom-get-methods">Creating Custom get*() Methods</h4>
You'll frequently find yourself wanting to query entities by some criteria besides Id.  For example, you might want to look up posts by title using a `getByTitle()` method.  Let's create an interface with this method:

```php
namespace MyApp\Posts\Orm\DataMappers;

use Opulence\Orm\DataMappers\IDataMapper;

interface IPostDataMapper extends IDataMapper
{
    public function getByTitle($title);
}
```

We'll implement this interface in the examples below.

<h2 id="sql-data-mappers">SQL Data Mappers</h2>
SQL data mappers use an SQL database for storage and querying.  `Opulence\Orm\DataMappers\SqlDataMapper` comes built-in.  The SQL data mapper does not automatically generate queries for you based on your table schema or your entities.  This is because doing so would enforce a particular schema on you when you may have already-existing tables set up.

`SqlDataMapper` comes with a few extra methods built-in:

* `loadEntity()`
  * Accepts a row of data from an SQL query and converts it into an instance of the object maintained by the data mapper
  * You must implement this for each SQL data mapper
* `read()`
  * Executes the input query and parameters and loads the data into objects using the `loadEntity()` method
  * Useful for get*() methods

<h4 id="sql-example">Example</h4>
Let's take a look at an example of an SQL data mapper for WordPress posts:

```php
namespace MyApp\WordPress\Orm\DataMappers;

use MyApp\WordPress\Post;
use Opulence\Orm\DataMappers\SqlDataMapper;
use PDO;

class PostSqlDataMapper extends SqlDataMapper implements IPostDataMapper
{
    /** @var Post $post */
    public function add($post)
    {
        $statement = $this->writeConnection->prepare(
            'INSERT INTO posts (text, title, author) VALUES (:text, :title, :author)'
        );
        $statement->bindValues([
            'text' => $post->getText(),
            'title' => $post->getTitle(),
            'author' => $post->getAuthor()
        ]);
        $statement->execute();
    }

    /** @var Post $post */
    public function delete($post)
    {
        $statement = $this->writeConnection->prepare(
            'DELETE FROM posts WHERE id = :id'
        );
        $statement->bindValues([
            'id' => [$post->getId(), \PDO::PARAM_INT]
        ]);
        $statement->execute();
    }

    public function getAll() : array
    {
        $sql = 'SELECT id, text, title, author FROM posts';

        // The last parameter says that we want a list of entities
        return $this->read($sql, [], self::VALUE_TYPE_ARRAY);
    }

    public function getById($id)
    {
        $sql = 'SELECT id, text, title, author FROM posts WHERE id = :id';
        $parameters = [
            'id' => [$id, \PDO::PARAM_INT]
        ];

        // The second-to-last parameter says that we want a single entity
        // The last parameter says that we expect one and only one entity
        return $this->read($sql, $parameters, self::VALUE_TYPE_ENTITY, true);
    }

    // This is a custom get*() method defined in IPostDataMapper
    public function getByTitle($title)
    {
        $sql = 'SELECT id, text, title, author FROM posts WHERE title = :title';
        $parameters = [
            'title' => $title
        ];

        return $this->read($sql, $parameters, self::VALUE_TYPE_ENTITY);
    }

    /** @var Post $post */
    public function update($post)
    {
        $statement = $this->writeConnection->prepare(
            'UPDATE posts SET text = :text, title = :title, author = :author WHERE id = :id'
        );
        $statement->bindValues([
            'text' => $post->getText(),
            'title' => $post->getTitle(),
            'author' => $post->getAuthor(),
            'id' => [$post->getId(), \PDO::PARAM_INT]
        ]);
        $statement->execute();
    }

    protected function loadEntity(array $hash)
    {
        // $hash contains the column values from a single row in the results
        return new Post(
            (int)$hash['id'],
            $hash['title'],
            $hash['author'],
            $hash['text']
        );
    }
}
```

> **Note:** The unit of work automatically handles setting the Id on the entity after the `add()` method is run as well as resetting it in case the transaction is rolled back.

<h4 id="query-builders">Query Builders</h4>
To simplify building your SQL queries, try Opulence's [query builders](database-query-builders).  For example, we can use a fluent syntax to define the `add()` method from the [above example](#sql-example).

```php
public function add($post)
{
    $query = (new \Opulence\QueryBuilders\PostgreSql\QueryBuilder)
        ->insert('posts', [
            'text' => $post->getText(),
            'title' => $post->getTitle(),
            'author' => $post->getAuthor()
        ]);
    $statement = $this->writeConnection->prepare($query->getSql());
    $statement->bindValues($query->getParameters());
    $statement->execute();
}
```

<h2 id="cache-data-mappers">Cache Data Mappers</h2>
Cache data mappers use some form of cache (eg Redis or Memcached) for storage.  They must implement the `Opulence\Orm\DataMappers\ICacheDataMapper` (`PhpRedisDataMapper` and `PredisDataMapper` come built-in).  A cache data mapper implements all the methods from `IDataMapper` as well as `flush()`, which flushes from cache all instances managed by the data mapper.

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
Let's take a look at a PhpRedis data mapper example:

```php
namespace MyApp\WordPress\Orm\DataMappers;

use MyApp\WordPress\Post;
use Opulence\Orm\DataMappers\PhpRedisDataMapper;
use Opulence\Orm\OrmException;

class PostRedisDataMapper extends PhpRedisDataMapper implements IPostDataMapper
{
    /** @var Post $post */
    public function add($post)
    {
        // Store a hash of the post's data
        $hash = [
            'id' => $post->getId(),
            'title' => $post->getTitle(),
            'author' => $post->getAuthor()
        ];

        if (!$this->redis->hMset("posts:{$post->getId()}", $hash)) {
            throw new OrmException('Failed to add post to Redis');
        }

        // Create an index of post Ids
        // This is useful for the getAll() method
        $this->redis->sAdd('posts', $post->getId());
        // Create an index of post titles
        // This is useful for the getByTitle() method
        $this->redis->set("posts:titles:{$post->getTitle()}", $post->getId());
    }

    /** @var Post $post */
    public function delete($post)
    {
        if (!$this->redis->del("posts:{$post->getId()}")) {
            throw new OrmException('Failed to delete post from Redis');
        }

        // Remove this post from the index of posts
        $this->redis->sRemove('posts', $post->getId());
    }

    public function flush()
    {
        // Delete the index of posts as well as all posts
        if (
            $this->redis->del('posts') === false ||
            !$this->redis->deleteKeyPatterns('posts:*')
        ) {
            throw new OrmException('Failed to flush posts from Redis');
        }
    }

    public function getAll() : array
    {
        // This will load all the Ids in the "posts" set and generate Post objects
        return $this->read('posts', self::VALUE_TYPE_SET);
    }

    // This is a custom get*() method defined in IPostDataMapper
    public function getByTitle($title)
    {
        // This will load the Id in the "posts:titles:$title" key and generate a Post object
        return $this->read("posts:titles:$title", self::VALUE_TYPE_STRING);
    }

    /** @var Post $post */
    public function update($post)
    {
        // In this case, an update will do the same thing as an addition
        $this->add($post);
    }

    protected function getEntityHashById($id)
    {
        $hash = $this->redis->hGetAll("posts:$id");

        if ($hash == []) {
            // In the case that nothing was found, return null
            return null;
        }

        return $hash;
    }

    protected function loadEntity(array $hash)
    {
        return new Post(
            (int)$hash['id'],
            $hash['title'],
            $hash['author']
        );
    }
}
```

<h2 id="cached-sql-data-mappers">Cached SQL Data Mappers</h2>
Cached SQL data mappers use an SQL database with a cache layer on top.  This reduces the number of queries going to your database by up to 95%, which drastically increases the speed of your data retrieval and improves scalability.  All writes are coordinated by the [unit of work](orm-units-of-work) so that cache and the SQL database are never out of sync.  Cached SQL data mappers contain a cache data mapper and an SQL data mapper.  It coordinates reads and writes between the two sub-data mappers and gives priority to the cache data mapper.  If data cannot be found in cache, it is queried from the SQL database and written back to cache for future queries.

> **Note:** The cache and SQL data mappers MUST implement the same interface for all `get*()` methods.  This allows the cached SQL data mapper to try and call a method from the cache data mapper, and if it fails, call it from the SQL data mapper.  If one of your data mappers does not return data for a particular method, just return `null` if the method returns a single entity, otherwise an empty array if it returns a list of entities.

Cached SQL data mappers must implement `Opulence\Orm\DataMappers\ICachedSqlDataMapper` (`CachedSqlDataMapper` comes built-in).  `RedisCachedSqlDataMapper` and `MemcachedCachedSqlDataMapper` both extend `CachedSqlDataMapper` and provide functionality to simplify interactions with Redis and Memcached, respectively.

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
* `setSqlDataMapper()`
  * Sets the instance of the sub-SQL data mapper using the input connection instances


<h4 id="cached-sql-example">Example</h4>
Let's take a look at a cached SQL data mapper example that uses the cache and SQL data mappers from the previous examples:

```php
namespace MyApp\WordPress\Orm\DataMappers;

use Opulence\Databases\IConnection;
use Opulence\Orm\DataMappers\RedisCachedSqlDataMapper;

class PostCachedSqlDataMapper extends RedisCachedSqlDataMapper implements IPostDataMapper
{
    public function getByTitle($title)
    {
        return $this->read('getByTitle', [$title]);
    }

    protected function setCacheDataMapper($cache)
    {
        $this->cacheDataMapper = new PostCacheDataMapper($cache);
    }

    protected function setSqlDataMapper(IConnection $readConnection, IConnection $writeConnection)
    {
        $this->sqlDataMapper = new PostSqlDataMapper($readConnection, $writeConnection);
    }
}
```

Our `getByTitle()` method calls `$this->read()`, which automatically handles reading from cache and falling back to the SQL database on a cache miss.  This is all you need to do to take advantage of aggressive caching in your data mappers.

<h2 id="relationships">Relationships</h2>
Instead of just containing the author's name, let's say your `Post` object contains an `Author` object.  Whenever you query a `Post` object from the data mapper, you'll also need to query the `Author` object.  The easiest way to do this is to inject the author repository into the post data mapper:

```php
use MyApp\WordPress\Orm\AuthorRepo;
use MyApp\WordPress\Post;

class PostDataMapper extends SqlDataMapper
{
    private $authorRepo;

    public function __construct(
        IConnection $readConnection,
        IConnection $writeConnection,
        AuthorRepo $authorRepo
    ) {
        parent::__construct($readConnection, $writeConnection);

        $this->authorRepo = $authorRepo;
    }

    protected function loadEntity(array $hash)
    {
        // Grab the author
        $author = $this->authorRepo->getById($hash['author_id']);

        return new Post(
            (int)$hash['id'],
            $hash['title'],
            $author
        );
    }
}
```

<h2 id="generating-data-mapper-classes">Generating Data Mapper Classes</h2>
You can use the console to generate any type of built-in data mapper using `php apex make:datamapper`, and then selecting from the menu.
