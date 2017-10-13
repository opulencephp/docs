# Collections

## Table of Contents
1. [Introduction](#introduction)
2. [Array Lists](#array-lists)
    1. [Adding Values](#array-lists-adding-values)
    2. [Getting Values](#array-lists-getting-values)
    3. [Checking for Values](#array-lists-checking-for-values)
    4. [Getting Indices](#array-lists-getting-indices)
    5. [Getting the Number of Values](#array-lists-getting-number-of-values)
    6. [Removing Values](#array-lists-removing-values)
    7. [Unioning Values](#array-lists-unioning-values)
    8. [Intersecting Values](#array-lists-intersecting-values)
    9. [Reversing the Values](#array-lists-reversing-values)
    10. [Sorting Values](#array-lists-sorting-values)
3. [Hash Tables](#hash-tables)
    1. [Adding Values](#hash-tables-adding-values)
    2. [Getting Values](#hash-tables-getting-values)
    3. [Checking for Values](#hash-tables-checking-for-values)
    4. [Getting the Number of Values](#hash-tables-getting-number-of-values)
    5. [Removing Values](#hash-tables-removing-values)
    6. [Unioning Values](#hash-tables-unioning-values)
    6. [Intersecting Values](#hash-tables-intersecting-values)
4. [Hash Sets](#hash-sets)
    1. [Adding Values](#hash-sets-adding-values)
    2. [Checking for Values](#hash-sets-checking-for-values)
    3. [Getting the Number of Values](#hash-sets-getting-number-of-values)
    4. [Removing Values](#hash-sets-removing-values)
    5. [Unioning Values](#hash-sets-unioning-values)
    6. [Intersecting Values](#hash-sets-intersecting-values)
    7. [Sorting Values](#hash-sets-sorting-values)
5. [Stacks](#stacks)
    1. [Pushing Values](#stacks-pushing-values)
    2. [Popping Values](#stacks-popping-values)
    3. [Peeking at Values](#stacks-peeking-at-values)
    4. [Checking for Values](#stacks-checking-for-values)
    5. [Getting the Number of Values](#stacks-getting-number-of-values)
    6. [Clearing Values](#stacks-clearing-values)
6. [Queues](#queues)
    1. [Enqueueing Values](#queues-enqueueing-values)
    2. [Dequeueing Values](#queues-dequeueing-values)
    3. [Peeking at Values](#queues-peeking-at-values)
    4. [Checking for Values](#queues-checking-for-values)
    5. [Getting the Number of Values](#queues-getting-number-of-values)
    6. [Clearing Values](#queues-clearing-values)
7. [Immutable Array Lists](#immutable-array-lists)
    1. [Getting Values](#immutable-array-lists-getting-values)
    2. [Checking for Values](#immutable-array-lists-checking-for-values)
    3. [Getting Indices](#immutable-array-lists-getting-indices)
    4. [Getting the Number of Values](#immutable-array-lists-getting-number-of-values)
8. [Immutable Hash Tables](#immutable-hash-tables)
    1. [Getting Values](#immutable-hash-tables-getting-values)
    2. [Checking for Values](#immutable-hash-tables-checking-for-values)
    3. [Getting the Number of Values](#immutable-hash-tables-getting-number-of-values)
9. [Immutable Hash Sets](#immutable-hash-sets)
    1. [Checking for Values](#immutable-hash-sets-checking-for-values)
    2. [Getting the Number of Values](#immutable-hash-sets-getting-number-of-values)

<h2 id="introduction">Introduction</h2>

Unfortunately, PHP's support for collections is relatively incomplete.  The `array` type is reused for multiple types, like hash tables and lists.  PHP also has _some_ support for advanced types in the SPL library, but it is incomplete, and its syntax is somewhat clunky.  To cover for PHP's lack of coverage of collections, Opulence provides simple wrappers for common collections found in most other programming languages.

<h2 id="array-lists">Array Lists</h2>

Opulence's `ArrayList` is probably the most similar to PHP's built-in indexed array functionality.  You can instantiate one with or without an array:

```php
use Opulence\Collections\ArrayList;

$arrayList = new ArrayList();
// Or...
$arrayList = new ArrayList(['foo', 'bar']);
```

If you want to grab the underlying array, call

```php
$array = $arrayList->toArray();
```

> **Note:** `ArrayList` implements `ArrayAccess` and `IteratorAggregate`, so you can use array-like accessors and iterate over it.

<h4 id="array-lists-adding-values">Adding Values</h4>

You can add a value via

```php
$arrayList->add('foo');
```

Or, you can add multiple values at once:

```php
$arrayList->addRange(['foo', 'bar']);
```

To insert a value at a specific index, call

```php
$arrayList->insert(23, 'foo');
```

<h4 id="array-lists-getting-values">Getting Values</h4>

To get the value at a certain index from an array list, call

```php
$value = $arrayList->get(123);
```

<h4 id="array-lists-checking-for-values">Checking for Values</h4>

To check for a value, call

```php
$containsValue = $arrayList->containsValue('foo');
```

<h4 id="array-lists-getting-indices">Getting Indices</h4>

To grab the index for a value, call

```php
$index = $arrayList->indexOf('foo');
```

If the array list doesn't contain the value, `null` will be returned.

<h4 id="array-lists-getting-number-of-values">Getting the Number of Values</h4>

To grab the number of values in the array list, call

```php
$count = $arrayList->count();
```

<h4 id="array-lists-removing-values">Removing Values</h4>

To remove a value by index, call

```php
$arrayList->removeIndex(123);
```

To remove a specific value, call

```php
$arrayList->removeValue('foo');
```

To remove all values, call

```php
$arrayList->clear();
```

<h4 id="array-lists-unioning-values">Unioning Values</h4>

You can union an array list's values with an array via

```php
$arrayList->union(['foo', 'bar']);
```

<h4 id="array-lists-intersecting-values">Intersecting Values</h4>

You can intersect an array list's values with an array by calling

```php
$arrayList->intersect(['foo', 'bar']);
```

<h4 id="array-lists-reversing-values">Reversing Values</h4>

To reverse the values in the list, call

```php
$arrayList->reverse();
```

<h4 id="array-lists-sorting-values">Sorting Values</h4>

You can sort values similar to the way you can sort PHP arrays via `usort()`:

```php
$comparer = function ($a, $b) {
    return $a > $b ? 1 : -1;
};
$arrayList->sort($comparer);
```

<h2 id="hash-tables">Hash Tables</h2>

Hash tables are most similar to PHP's built-in associative array functionality.  It maps a key to a value.  You can instantiate it with or without an array of key => value pairs:

```php
use Opulence\Collections\HashTable;

$hashTable = new HashTable();
```

Or...

```php
$hashTable = new HashTable(['foo' => 'bar']);
```

To get the underlying array, call

```php
$array = $hashTable->toArray();
```

> **Note:** `HashTable` implements `ArrayAccess` and `IteratorAggregate`, so you can use array-like accessors and iterate over it.

<h4 id="hash-tables-adding-values">Adding Values</h4>

To add a value, call

```php
$hashTable->add('foo', 'bar');
```

<h4 id="hash-tables-getting-values">Getting Values</h4>

To get a value at a key, call

```php
$value = $hashTable->get('foo');
```

<h4 id="hash-tables-checking-for-values">Checking for Values</h4>

To check for a key, call

```php
$containsKey = $hashTable->containsKey('foo');
```

To check for a value, call

```php
$containsValue = $hashTable->containsValue('foo');
```

<h4 id="hash-tables-getting-number-of-values">Getting the Number of Values</h4>

To get the number of values in the hash table, call

```php
$count = $hashTable->count();
```

<h4 id="hash-tables-removing-values">Removing Values</h4>

To remove a value at a certain key, call

```php
$hashTable->removeKey('foo');
```

<h4 id="hash-tables-unioning-values">Unioning Values</h4>

You can union a set's values with an array via

```php
$hashTable->union(['foo' => 'bar']);
```

<h4 id="hash-tables-intersecting-values">Intersecting Values</h4>

You can intersect a set's values with an array by calling

```php
$hashTable->intersect(['foo' => 'bar']);
```

<h2 id="hash-sets">Hash Sets</h2>

Hash sets are lists with unique values.  You can instantiate one with or without an array of key => value pairs:

```php
use Opulence\Collections\HashSet;

$set = new HashSet();
```

Or...

```php
$set = new HashSet(['foo', 'bar']);
```

To get the underlying array, call

```php
$array = $set->toArray();
```

> **Note:** `HashSet` implements `ArrayAccess` and `IteratorAggregate`, so you can use array-like accessors and iterate over it.

<h4 id="hash-sets-adding-values">Adding Values</h4>

You can add a value via

```php
$set->add('foo');
```

Or, you can add multiple values at once:

```php
$set->addRange(['foo', 'bar']);
```

<h4 id="hash-sets-checking-for-values">Checking for Values</h4>

To check for a value, call

```php
$containsValue = $set->containsValue('foo');
```

<h4 id="hash-sets-getting-number-of-values">Getting the Number of Values</h4>

To grab the number of values in the hash set, call

```php
$count = $set->count();
```

<h4 id="hash-sets-removing-values">Removing Values</h4>

To remove a specific value, call

```php
$set->removeValue('foo');
```

To remove all values, call

```php
$set->clear();
```

<h4 id="hash-sets-unioning-values">Unioning Values</h4>

You can union a hash set with an array via

```php
$set->union(['foo', 'bar']);
```

<h4 id="hash-sets-intersecting-values">Intersecting Values</h4>

You can intersect a hash set with an array by calling

```php
$set->intersect(['foo', 'bar']);
```

<h4 id="hash-sets-sorting-values">Sorting Values</h4>

You can sort values similar to the way you can sort PHP arrays via `usort()`:

```php
$comparer = function ($a, $b) {
    return $a > $b ? 1 : -1;
};
$set->sort($comparer);
```

<h2 id="stacks">Stacks</h2>

Stacks are first-in, last-out (FILO) data structures.  To create one, call

```php
use Opulence\Collections\Stack;

$stack = new Stack();
```

To get the underlying array, call

```php
$array = $stack->toArray();
```

> **Note:** `Stack` implements `IteratorAggregate`, so you can iterate over it.

<h4 id="stacks-pushing-values">Pushing Values</h4>

To push a value onto the stack, call

```php
$stack->push('foo');
```

<h4 id="stacks-popping-values">Popping Values</h4>

To pop a value off the stack, call

```php
$value = $stack->pop();
```

If there are no values in the stack, this will return `null`.

<h4 id="stacks-peeking-at-values">Peeking at Values</h4>

To peek at the top value in the stack, call

```php
$value = $stack->peek();
```

If there are no values in the stack, this will return `null`.

<h4 id="stacks-checking-for-values">Checking for Values</h4>

To check for a value within a stack, call

```php
$containsValue = $stack->containsValue('foo');
```

<h4 id="stacks-getting-number-of-values">Getting the Number of Values</h4>

To get the number of values in the stack, call

```php
$count = $stack->count();
```

<h4 id="stacks-clearing-values">Clearing Values</h4>

To clear the stack, call

```php
$stack->clear();
```

<h2 id="queues">Queues</h2>

Queues are first-in, first-out (FIFO) data structures.  To create one, call

```php
use Opulence\Collections\Queue;

$queue = new Queue();
```

To get the underlying array, call

```php
$array = $queue->toArray();
```

> **Note:** `Queue` implements `IteratorAggregate`, so you can iterate over it.

<h4 id="queues-enqueueing-values">Enqueueing Values</h4>

To enqueue a value onto the queue, call

```php
$queue->enqueue('foo');
```

<h4 id="queues-dequeueing-values">Dequeueing Values</h4>

To dequeue a value from the queue, call

```php
$value = $queue->dequeue();
```

If there are no values in the queue, this will return `null`.

<h4 id="queues-peeking-at-values">Peeking at Values</h4>

To peek at the value at the beginning of the queue, call

```php
$value = $queue->peek();
```

If there are no values in the queue, this will return `null`.

<h4 id="queues-checking-for-values">Checking for Values</h4>

To check for a value within a queue, call

```php
$containsValue = $queue->containsValue('foo');
```

<h4 id="queues-getting-number-of-values">Getting the Number of Values</h4>

To get the number of values in the queue, call

```php
$count = $queue->count();
```

<h4 id="queues-clearing-values">Clearing Values</h4>

To clear the queue, call

```php
$queue->clear();
```

<h2 id="immutable-array-lists">Immutable Array Lists</h2>

`ImmutableArrayList` are read-only [array lists](#array-lists).  To instantiate one, pass in the array of values:

```php
use Opulence\Collections\ImmutableArrayList;

$arrayList = new ImmutableArrayList(['foo', 'bar']);
```

If you want to grab the underlying array, call

```php
$array = $arrayList->toArray();
```

> **Note:** `ImmutableArrayList` implements `ArrayAccess` and `IteratorAggregate`, so you can use array-like accessors and iterate over it.

<h4 id="immutable-array-lists-getting-values">Getting Values</h4>

To get the value at a certain index from an array list, call

```php
$value = $arrayList->get(123);
```

<h4 id="immutable-array-lists-checking-for-values">Checking for Values</h4>

To check for a value, call

```php
$containsValue = $arrayList->containsValue('foo');
```

<h4 id="immutable-array-lists-getting-indices">Getting Indices</h4>

To grab the index for a value, call

```php
$index = $arrayList->indexOf('foo');
```

If the array list doesn't contain the value, `null` will be returned.

<h4 id="immutable-array-lists-getting-number-of-values">Getting the Number of Values</h4>

To grab the number of values in the array list, call

```php
$count = $arrayList->count();
```

<h2 id="immutable-hash-tables">Immutable Hash Tables</h2>

Sometimes, your business logic might dictate that a hash table is read-only.  Opulence provides support via `ImmutableHashTable`.  It requires that you pass values into its constructor:

```php
use Opulence\Collections\ImmutableHashTable;

$hashTable = new ImmutableHashTable(['foo' => 'bar', 'baz' => 'blah']);
```

To get the underlying array, call

```php
$array = $hashTable->toArray();
```

> **Note:** `ImmutableHashTable` implements `ArrayAccess` and `IteratorAggregate`, so you can use array-like accessors and iterate over it.

<h4 id="immutable-hash-tables-getting-values">Getting Values</h4>

To get a value at a key, call

```php
$value = $hashTable->get('foo');
```

<h4 id="immutable-hash-tables-checking-for-values">Checking for Values</h4>

To check for a key, call

```php
$containsKey = $hashTable->containsKey('foo');
```

To check for a value, call

```php
$containsValue = $hashTable->containsValue('foo');
```

<h4 id="immutable-hash-tables-getting-number-of-values">Getting the Number of Values</h4>

To get the number of values in the hash table, call

```php
$count = $hashTable->count();
```

<h2 id="immutable-hash-sets">Immutable Hash Sets</h2>

Immutable hash sets are read-only [sets](#sets).  You can instantiate one with a list of values:

```php
use Opulence\Collections\ImmutableHashSet;

$set = new ImmutableHashSet(['foo', 'bar']);
```

To get the underlying array, call

```php
$array = $set->toArray();
```

> **Note:** `ImmutableHashSet` implements `ArrayAccess` and `IteratorAggregate`, so you can use array-like accessors and iterate over it.

<h4 id="immutable-hash-sets-checking-for-values">Checking for Values</h4>

To check for a value, call

```php
$containsValue = $set->containsValue('foo');
```

<h4 id="immutable-hash-sets-getting-number-of-values">Getting the Number of Values</h4>

To grab the number of values in the set, call

```php
$count = $set->count();
```