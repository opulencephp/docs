# Repositories

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Usage](#basic-usage)
  1. [Adding Entities](#add)
  2. [Deleting Entities](#delete)
  3. [Getting All Entities](#get-all)
  4. [Getting By Id](#get-by-id)
3. [Committing Changes](#commit)
4. [Updating Entities](#update)

<h2 id="introduction">Introduction</h2>
**Repositories** are simply collections of entities.  They provide methods for adding, deleting, and retrieving entities, but they leave the actual data retrieval to [**data mappers**](orm-data-mappers).  These data mappers can interact with an SQL database, cache, some other form of storage, or a mixture of storage mechanisms.  By utilizing a [**unit of work**](orm-units-of-work), writes to the data mappers are scheduled and only executed when calling `$unitOfWork->commit()`.  This gives you the ability to wrap multiple repositories' writes into a single, "all-or-nothing" transaction.

<h2 id="basic-usage">Basic Usage</h2>
If your repository will not implement any methods outside of `Opulence\Orm\Repositories\Repository`, you don't even have to create your own repository class.  Just use `Opulence\Orm\Repositories\Repository`:

```php
use MyApp\WordPress\Post;

// Assume $dataMapper and $unitOfWork are already instantiated
$repo = new Repository(Post::class, $dataMapper, $unitOfWork);
```

However, if your repository implements any custom `get*()` methods, you'll have to extend `Opulence\Orm\Repositories\Repository`.  Let's take a look at a repository that supports a `getByTitle()` method:

```php
namespace MyApp\WordPress\Orm;

use Opulence\Orm\Repositories\Repository;

class PostRepo extends Repository
{
    public function getByTitle($title)
    {
        return $this->getFromDataMapper('getByTitle', [$title]);
    }
}
```

Rather than calling `$this->dataMapper->getByTitle()` directly, you should use the helper function `$this->getFromDataMapper()`, which automatically handles registering entities to the entity registry.  Next, you'll need to [add the `getByTitle()` method to your data mapper](orm-data-mappers#creating-custom-get-methods).

<h4 id="add">Adding Entities</h4>
```php
$postToAdd = new Post(123, 'First Post', 'This is my first post');
$repo->add($postToAdd);
```

The new post will be scheduled for insertion by the unit of work.

<h4 id="delete">Deleting Entities</h4>
```php
$postToDelete = new Post(123, 'First Post', 'This is my first post');
$repo->delete($postToDelete);
```

The post will be scheduled for deletion by the unit of work.

<h4 id="get-all">Getting All Entities</h4>
```php
$posts = $repo->getAll();

foreach ($posts as $post) {
    echo $post->getTitle() . '<br />';
}
```

The list of post titles will be printed to the screen.  Every entity returned by `getAll()` is registered to the unit of work's [entity registry](orm-units-of-work#entity-registry).

<h4 id="get-by-id">Getting By Id</h4>
```php
$post = $repo->getById(123);
echo $post->getTitle();
```

All entities returned by the repository are automatically registered to the unit of work's [entity registry](orm-units-of-work#entity-registry), effectively turning it into a cache.  The repository first looks for the entity in the registry and returns it if it's already registered.  Otherwise, the data mapper is called and the entity is registered for future queries.

> **Note:** If no entity is found, an `Opulence\Orm\OrmException` will be thrown.

<h2 id="commit">Committing Changes</h2>
As you can see, there is no `save()` method in repositories.  To actually save any writes made by the repository, you must call `commit()` on the unit of work passed into the repository's constructor:

```php
use MyApp\WordPress\Orm\DataMappers\PostSqlDataMapper;
use MyApp\WordPress\Post;
use Opulence\Orm\Repositories\Repository;
use Opulence\Orm\UnitOfWork;

$dataMapper = new PostSqlDataMapper();
// Assume $unitOfWork is already instantiated
$repo = new Repository(Post::class, $dataMapper, $unitOfWork);
$postToDelete = $repo->getById(123);
$repo->delete($postToDelete);
$unitOfWork->commit();
```

Whenever you call `Repository::add()` or `Repository::delete()`, the entity is scheduled to be written by the data mappers back to storage.  Calling `$unitOfWork->commit()` actually writes all the scheduled entities to storage.

<h2 id="update">Updating Entities</h2>
Updates are automatically tracked by the [entity registry](orm-units-of-work#entity-registry).  For example, let's say we change a post's title:

```php
$post = $repo->getById(123);
$post->setTitle('Better Title');
$unitOfWork->commit();
```

The entity registry will determine that `$post` has been updated, and it will schedule the post to be updated in storage by the data mappers.
