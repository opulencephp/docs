# Unit of Work

## Table of Contents
1. [Introduction](#introduction)
2. [Example](#example)
3. [Entity Registry](#entity-registry)
4. [Aggregate Roots](#aggregate-roots)

<h2 id="introduction">Introduction</h2>
**Units of work** act as transactions across multiple repositories.  They also schedule entity updates/insertions/deletions in the data mappers.  The benefits of using units of work include:
                                                                                                                                             
1. Transactions across multiple repositories can be rolled back, giving you "all or nothing" functionality
2. Changes made to entities retrieved by repositories are automatically checked for changes and, if any are found, scheduled for updating when the unit of work is committed
3. Database writes are queued and executed all at once when the unit of work is committed, giving you better performance than executing writes throughout the lifetime of the application
4. Querying for the same object will always give you the same, single instance of that object
 
<h2 id="example">Example</h2>
 Let's take a look at how units of work can manage entities retrieved through repositories:
```php
use MyApp\ORM\DataMappers\MyDataMapper;
use Opulence\ORM\EntityRegistry;
use Opulence\ORM\Repositories\Repo;
use Opulence\ORM\UnitOfWork;

// Assume $connection was set previously
$unitOfWork = new UnitOfWork(new EntityRegistry(), $connection);
$dataMapper = new MyDataMapper();
$users = new Repo("Opulence\\Users\\User", $dataMapper, $unitOfWork);

// Let's say we know that there's a user with Id 123 and username "foo" in the repository
$someUser = $users->getById(123);
echo $someUser->getUsername(); // "foo"

// Let's change his username
$someUser->setUsername("bar");
// Once we're done with our unit of work, just let it know you're ready to commit
// It'll automatically know what has changed and save those changes back to storage
$unitOfWork->commit();

// To prove that this really worked, let's print the name of the user now
echo $users->getById(123)->getUsername(); // "bar"
```

<h2 id="entity-registry">Entity Registry</h2>
Entities that are scheduled for insertion/deletion/update are managed by an `EntityRegistry`.  The `EntityRegistry` is also responsible for tracking any changes made to the entities it manages.  By default, it uses reflection, which for some classes might be slow.  To speed up the comparison between two objects to see if they're identical, you can use `registerComparisonFunction()`.

Let's say that all you care about when checking if two users are identical is whether or not their usernames are identical:

```php
use Opulence\ORM\EntityRegistry;
use Opulence\ORM\UnitOfWork;

// Assume $connection was set previously
// Also assume the user object was already instantiated
$entityRegistry = new EntityRegistry();
$unitOfWork = new UnitOfWork($entityRegistry, $connection);
$className = $entityRegistry->getClassName($user);
$entityRegistry->manageEntity($user);
$user->setUsername("newUsername");

// Register a function that compares the usernames of two user objects
$entityRegistry->registerComparisonFunction($className, function($userA, $userB)
{
    return $userA->getUsername() == $userB->getUsername();
});

// On commit, the entity registry will run the comparison function
// It will determine that the $user's username has changed
// So, it will be scheduled for update and committed
$unitOfWork->commit();
```
> **Note:** PHP's `clone` feature performs a shallow clone.  In other words, it only clones the object, but not any objects contained in that object.  If your object contains another object and you'd like to take advantage of automatic change tracking, you must write a `__clone()` method for that class to clone any objects it contains.  Otherwise, the automatic change tracking will not pick up on changes made to the objects contained in other objects.

<h2 id="aggregate-roots">Aggregate Roots</h2>
Let's say that when creating a user you also create a password object.  This password object has a reference to the user object's Id.  In this case, the user is what we call an **aggregate root** because without it, the password wouldn't exist.  It'd be perfectly reasonable to insert both of them in the same unit of work.  However, if you did this, you might be asking yourself "How do I get the Id of the user before storing the password?"  The answer is `registerAggregateRootChild()`:
```php
// Order here matters: aggregate roots should be added before their children
$unitOfWork->scheduleForInsertion($user);
$unitOfWork->scheduleForInsertion($password);

// Pass in the aggregate root, the child, and the function that sets the aggregate root Id
$unitOfWork->registerAggregateRootChild($user, $password, function($user, $password)
{
    // This will be executed after the user is inserted but before the password is inserted
    $password->setUserId($user->getId());
});

$unitOfWork->commit();
echo $password->getUserId() == $user->getId(); // 1
```

> **Note:** Aggregate root functions are executed for entities scheduled for insertion and update.