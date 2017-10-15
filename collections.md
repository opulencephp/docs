# Collections

## Table of Contents
1. [Introduction](#introduction)
2. [Key-Value Pairs](#key-value-pairs)
    1. [getKey()](#key-value-pairs-getting-keys)
    2. [getValue()](#key-value-pairs-get)
3. [Array Lists](#array-lists)
    1. [add()](#array-lists-add)
    2. [addRange()](#array-lists-add-range)
    3. [clear()](#array-lists-clear)
    4. [containsValue()](#array-lists-contains-value)
    5. [count()](#array-lists-count)
    6. [get()](#array-lists-get)
    7. [indexOf()](#array-lists-index-of)
    8. [insert()](#array-lists-insert)
    9. [intersect()](#array-lists-intersect)
    10. [removeIndex()](#array-lists-remove-value)
    12. [reverse()](#array-lists-reverse)
    13. [sort()](#array-lists-sort)
    14. [toArray()](#array-lists-to-array)
    15. [union()](#array-lists-union)
4. [Hash Tables](#hash-tables)
    1. [add()](#hash-tables-add)
    2. [addRange()](#hash-tables-add-range)
    3. [clear()](#hash-tables-clear)
    4. [containsKey()](#hash-tables-contains-key)
    5. [containsValue()](#hash-tables-contains-value)
    6. [count()](#hash-tables-count)
    7. [get()](#hash-tables-get)
    8. [removeKey()](#hash-tables-remove-key)
    9. [removeValue()](#hash-tables-remove-value)
    10. [toArray()](#hash-tables-to-array)
5. [Hash Sets](#hash-sets)
    1. [add()](#hash-sets-add)
    2. [addRange()](#hash-sets-add-range)
    3. [clear()](#hash-sets-clear)
    4. [containsValue()](#hash-sets-contains-value)
    5. [count()](#hash-sets-count)
    6. [intersect()](#hash-sets-intersect)
    7. [sort()](#hash-sets-sort)
    8. [removeValue()](#hash-sets-remove-value)
    9. [toArray()](#hash-sets-to-array)
    10. [union()](#hash-sets-union)
6. [Stacks](#stacks)
    1. [clear()](#stacks-clear)
    2. [containsValue()](#stacks-contains-value)
    3. [count()](#stacks-count)
    4. [peek()](#stacks-peek)
    5. [pop()](#stacks-pop)
    6. [push()](#stacks-push)
    7. [toArray()](#stacks-to-array)
7. [Queues](#queues)
    1. [clear()](#queues-clear)
    2. [containsValue()](#queues-contains-value)
    3. [count()](#queues-count)
    4. [dequeue()](#queues-dequeue)
    5. [enqueue()](#queues-enqueue)
    6. [peek()](#queues-peek)
    7. [toArray()](#queues-to-array)
8. [Immutable Array Lists](#immutable-array-lists)
    1. [containsValue()](#immutable-array-lists-contains-value)
    2. [count()](#immutable-array-lists-count)
    3. [get()](#immutable-array-lists-get)
    4. [indexOf()](#immutable-array-lists-index-of)
    5. [toArray()](#immutable-array-lists-to-array)
9. [Immutable Hash Tables](#immutable-hash-tables)
    1. [containsValue()](#immutable-hash-tables-contains-value)
    2. [count()](#immutable-hash-tables-count)
    3. [get()](#immutable-hash-tables-get)
    4. [toArray()](#immutable-hash-tables-to-array)
10. [Immutable Hash Sets](#immutable-hash-sets)
    1. [containsValue()](#immutable-hash-sets-contains-value)
    2. [count()](#immutable-hash-sets-count)
    3. [toArray()](#immutable-hash-sets-to-array)

<h2 id="introduction">Introduction</h2>

Unfortunately, PHP's support for collections is relatively incomplete.  The `array` type is reused for multiple types, like hash tables and lists.  PHP also has _some_ support for advanced types in the SPL library, but it is incomplete, and its syntax is somewhat clunky.  To cover for PHP's lack of coverage of collections, Opulence provides simple wrappers for common collections found in most other programming languages.

<h2 id="key-value-pairs">Key-Value Pairs</h2>

Like its name implies, `KeyValuePair` holds a key and a value.  An example usage of in Opulence's [`HashTable`](#hash-tables) and [`ImmutableHashTable` 
](#immutable-hash-tables).  To instantiate one, pass in the key and value:

```php
use Opulence\Collections\KeyValuePair;

$kvp = new KeyValuePair('thekey', 'thevalue');
```

<h4 id="key-value-pairs-getting-keys">`getKey()`</h4>

To get the key-value pair's key, call

```php
$kvp->getKey();
```

<h4 id="key-value-pairs-get">`getValue()`</h4>

To get the key-value pair's value, call

```php
$kvp->getValue();
```

<h2 id="array-lists">Array Lists</h2>

Opulence's `ArrayList` is probably the most similar to PHP's built-in indexed array functionality.  You can instantiate one with or without an array:

```php
use Opulence\Collections\ArrayList;

$arrayList = new ArrayList();
// Or...
$arrayList = new ArrayList(['foo', 'bar']);
```

> **Note:** `ArrayList` implements `ArrayAccess` and `IteratorAggregate`, so you can use array-like accessors and iterate over it.

<h4 id="array-lists-add">`add()`</h4>

You can add a value via

```php
$arrayList->add('foo');
```

<h4 id="array-lists-add-range">`addRange()`</h4>

You can add multiple values at once:

```php
$arrayList->addRange(['foo', 'bar']);
```

<h4 id="array-lists-clear">`clear()`</h4>

You can remove all values in the array list:

```php
$arrayList->clear();
```

<h4 id="array-lists-count">`count()`</h4>

To grab the number of values in the array list, call

```php
$count = $arrayList->count();
```

<h4 id="array-lists-get">`get()`</h4>

To get the value at a certain index from an array list, call

```php
$value = $arrayList->get(123);
```

<h4 id="array-lists-contains-value">`containsValue()`</h4>

To check for a value, call

```php
$containsValue = $arrayList->containsValue('foo');
```

<h4 id="array-lists-index-of">`indexOf()`</h4>

To grab the index for a value, call

```php
$index = $arrayList->indexOf('foo');
```

<h4 id="array-lists-insert">`insert()`</h4>

To insert a value at a specific index, call

```php
$arrayList->insert(23, 'foo');
```

<h4 id="array-lists-intersect">`intersect()`</h4>

You can intersect an array list's values with an array by calling

```php
$arrayList->intersect(['foo', 'bar']);
```

If the array list doesn't contain the value, `null` will be returned.

<h4 id="array-lists-remove-value">`removeIndex()`</h4>

To remove a value by index, call

```php
$arrayList->removeIndex(123);
```

To remove a specific value, call

```php
$arrayList->removeValue('foo');
```

<h4 id="array-lists-reverse">`reverse()`</h4>

To reverse the values in the list, call

```php
$arrayList->reverse();
```

<h4 id="array-lists-sort">`sort()`</h4>

You can sort values similar to the way you can sort PHP arrays via `usort()`:

```php
$comparer = function ($a, $b) {
    return $a > $b ? 1 : -1;
};
$arrayList->sort($comparer);
```

<h4 id="array-lists-to-array">`toArray()`</h4>

You can get the underlying array by calling

```php
$array = $arrayList->toArray();
```

<h4 id="array-lists-union">`union()`</h4>

You can union an array list's values with an array via

```php
$arrayList->union(['foo', 'bar']);
```

<h2 id="hash-tables">Hash Tables</h2>

Hash tables are most similar to PHP's built-in associative array functionality.  It maps a key to a value.  In hash tables, the keys can be scalars, objects, arrays, or resources; the values can be any type.  You can instantiate it with or without an array of key => value pairs:

```php
use Opulence\Collections\HashTable;

$hashTable = new HashTable();
```

Or...

```php
$hashTable = new HashTable(['foo' => 'bar']);
```

> **Note:** `HashTable` implements `ArrayAccess` and `IteratorAggregate`, so you can use array-like accessors and iterate over it.

<h4 id="hash-tables-add">`add()`</h4>

To add a value, call

```php
$hashTable->add('foo', 'bar');
```

<h4 id="hash-tables-add-range">`addRange()`</h4>

To add multiple values, call `addRange()`:

```php
$hashTable->addRange(['foo' => 'bar', 'baz' => 'blah']);
```

<h4 id="hash-tables-clear">`clear()`</h4>

You can remove all values:

```php
$hashTable->clear();
```

<h4 id="hash-tables-contains-key">`containsKey()`</h4>

To check for a value, call

```php
$containsValue = $hashTable->containsValue('foo');
```

<h4 id="hash-tables-contains-value">`containsValue()`</h4>

To check for a key, call

```php
$containsKey = $hashTable->containsKey('foo');
```

<h4 id="hash-tables-count">`count()`</h4>

To get the number of values in the hash table, call

```php
$count = $hashTable->count();
```

<h4 id="hash-tables-get">`get()`</h4>

To get a value at a key, call

```php
$value = $hashTable->get('foo');
```

<h4 id="hash-tables-remove-key">`removeKey()`</h4>

To remove a value at a certain key, call

```php
$hashTable->removeKey('foo');
```

<h4 id="hash-tables-remove-value">`removeValue()`</h4>

To remove a value, call

```php
$hashTable->removeValue('foo');
```

<h4 id="hash-tables-to-array">`toArray()`</h4>

To get the underlying array, call

```php
$array = $hashTable->toArray();
```

This will return a list of `KeyValuePair` - not an associative array.  The reason for this is that keys can be non-strings, which is not supported in PHP.

<h2 id="hash-sets">Hash Sets</h2>

Hash sets are lists with unique values.  They accept objects, scalars, arrays, and resources as values.  You can instantiate one with or without an array of key => value pairs:

```php
use Opulence\Collections\HashSet;

$set = new HashSet();
```

Or...

```php
$set = new HashSet(['foo', 'bar']);
```

> **Note:** `HashSet` implements `ArrayAccess` and `IteratorAggregate`, so you can use array-like accessors and iterate over it.

<h4 id="hash-sets-add">`add()`</h4>

You can add a value via

```php
$set->add('foo');
```

<h4 id="hash-sets-add-range">`addRange()`</h4>

You can add multiple values at once:

```php
$set->addRange(['foo', 'bar']);
```

<h4 id="hash-sets-clear">`clear()`</h4>

To remove all values in the set, call `clear()`:

```php
$set->clear();
```

<h4 id="hash-sets-contains-value">`containsValue()`</h4>

To check for a value, call

```php
$containsValue = $set->containsValue('foo');
```

<h4 id="hash-sets-count">`count()`</h4>

To grab the number of values in the hash set, call

```php
$count = $set->count();
```

<h4 id="hash-sets-intersect">`intersect()`</h4>

You can intersect a hash set with an array by calling

```php
$set->intersect(['foo', 'bar']);
```

<h4 id="hash-sets-remove-value">`removeValue()`</h4>

To remove a specific value, call

```php
$set->removeValue('foo');
```

<h4 id="hash-sets-sort">`sort()`</h4>

You can sort values similar to the way you can sort PHP arrays via `usort()`:

```php
$comparer = function ($a, $b) {
    return $a > $b ? 1 : -1;
};
$set->sort($comparer);
```

<h4 id="hash-sets-to-array">`toArray()`</h4>

To get the underlying array, call

```php
$array = $set->toArray();
```

<h4 id="hash-sets-union">`union()`</h4>

You can union a hash set with an array via

```php
$set->union(['foo', 'bar']);
```

<h2 id="stacks">Stacks</h2>

Stacks are first-in, last-out (FILO) data structures.  To create one, call

```php
use Opulence\Collections\Stack;

$stack = new Stack();
```

> **Note:** `Stack` implements `IteratorAggregate`, so you can iterate over it.

<h4 id="stacks-clear">`clear()`</h4>

To clear the values in the stack, call

```php
$stack->clear();
```

<h4 id="stacks-contains-value">`containsValue()`</h4>

To check for a value within a stack, call

```php
$containsValue = $stack->containsValue('foo');
```

<h4 id="stacks-count">`count()`</h4>

To get the number of values in the stack, call

```php
$count = $stack->count();
```

<h4 id="stacks-peek">`peek()`</h4>

To peek at the top value in the stack, call

```php
$value = $stack->peek();
```

<h4 id="stacks-pop">`pop()`</h4>

To pop a value off the stack, call

```php
$value = $stack->pop();
```

If there are no values in the stack, this will return `null`.

<h4 id="stacks-push">`push()`</h4>

To push a value onto the stack, call

```php
$stack->push('foo');
```

If there are no values in the stack, this will return `null`.

<h4 id="stacks-to-array">`toArray()`</h4>

To get the underlying array, call

```php
$array = $stack->toArray();
```

<h2 id="queues">Queues</h2>

Queues are first-in, first-out (FIFO) data structures.  To create one, call

```php
use Opulence\Collections\Queue;

$queue = new Queue();
```

> **Note:** `Queue` implements `IteratorAggregate`, so you can iterate over it.

<h4 id="queues-clear">`clear()`</h4>

To clear the queue, call

```php
$queue->clear();
```

<h4 id="queues-contains-value">`containsValue()`</h4>

To check for a value within a queue, call

```php
$containsValue = $queue->containsValue('foo');
```

<h4 id="queues-count">`count()`</h4>

To get the number of values in the queue, call

```php
$count = $queue->count();
```

<h4 id="queues-dequeue">`dequeue()`</h4>

To dequeue a value from the queue, call

```php
$value = $queue->dequeue();
```

If there are no values in the queue, this will return `null`.

<h4 id="queues-enqueue">`enqueue()`</h4>

To enqueue a value onto the queue, call

```php
$queue->enqueue('foo');
```

<h4 id="queues-peek">`peek()`</h4>

To peek at the value at the beginning of the queue, call

```php
$value = $queue->peek();
```

If there are no values in the queue, this will return `null`.

<h4 id="queues-to-array">`toArray()`</h4>

To get the underlying array, call

```php
$array = $queue->toArray();
```

<h2 id="immutable-array-lists">Immutable Array Lists</h2>

`ImmutableArrayList` are read-only [array lists](#array-lists).  To instantiate one, pass in the array of values:

```php
use Opulence\Collections\ImmutableArrayList;

$arrayList = new ImmutableArrayList(['foo', 'bar']);
```

> **Note:** `ImmutableArrayList` implements `ArrayAccess` and `IteratorAggregate`, so you can use array-like accessors and iterate over it.

<h4 id="immutable-array-lists-contains-value">`containsValue()`</h4>

To check for a value, call

```php
$containsValue = $arrayList->containsValue('foo');
```

<h4 id="immutable-array-lists-count">`count()`</h4>

To grab the number of values in the array list, call

```php
$count = $arrayList->count();
```

<h4 id="immutable-array-lists-get">`get()`</h4>

To get the value at a certain index from an array list, call

```php
$value = $arrayList->get(123);
```

<h4 id="immutable-array-lists-index-of">`indexOf()`</h4>

To grab the index for a value, call

```php
$index = $arrayList->indexOf('foo');
```

If the array list doesn't contain the value, `null` will be returned.

<h4 id="immutable-array-lists-to-array">`toArray()`</h4>

If you want to grab the underlying array, call

```php
$array = $arrayList->toArray();
```

<h2 id="immutable-hash-tables">Immutable Hash Tables</h2>

Sometimes, your business logic might dictate that a [hash table](#hash-tables) is read-only.  Opulence provides support via `ImmutableHashTable`.  In immutable hash tables, the keys can be scalars, objects, arrays, or resources; the values can be any type.  It requires that you pass values into its constructor:

```php
use Opulence\Collections\ImmutableHashTable;

$hashTable = new ImmutableHashTable(['foo' => 'bar', 'baz' => 'blah']);
```

> **Note:** `ImmutableHashTable` implements `ArrayAccess` and `IteratorAggregate`, so you can use array-like accessors and iterate over it.

<h4 id="immutable-hash-tables-contains-value">`containsValue()`</h4>

To check for a key, call

```php
$containsKey = $hashTable->containsKey('foo');
```

To check for a value, call

```php
$containsValue = $hashTable->containsValue('foo');
```

<h4 id="immutable-hash-tables-count">`count()`</h4>

To get the number of values in the hash table, call

```php
$count = $hashTable->count();
```

<h4 id="immutable-hash-tables-get">`get()`</h4>

To get a value at a key, call

```php
$value = $hashTable->get('foo');
```

<h4 id="immutable-hash-tables-to-array">`toArray()`</h4>

To get the underlying array, call

```php
$array = $hashTable->toArray();
```

This will return a list of `KeyValuePair` - not an associative array.  The reason for this is that keys can be non-strings (eg objects) in hash tables, but keys in PHP associative arrays must be serializable.

<h2 id="immutable-hash-sets">Immutable Hash Sets</h2>

Immutable hash sets are read-only [hash sets](#hash-sets).  They accept objects, scalars, arrays, and resources as values.  You can instantiate one with a list of values:

```php
use Opulence\Collections\ImmutableHashSet;

$set = new ImmutableHashSet(['foo', 'bar']);
```

> **Note:** `ImmutableHashSet` implements `ArrayAccess` and `IteratorAggregate`, so you can use array-like accessors and iterate over it.

<h4 id="immutable-hash-sets-contains-value">`containsValue()`</h4>

To check for a value, call

```php
$containsValue = $set->containsValue('foo');
```

<h4 id="immutable-hash-sets-count">`count()`</h4>

To grab the number of values in the set, call

```php
$count = $set->count();
```

<h4 id="immutable-hash-sets-to-array">`toArray()`</h4>

To get the underlying array, call

```php
$array = $set->toArray();
```