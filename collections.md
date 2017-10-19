# Collections

## Table of Contents
1. [Introduction](#introduction)
2. [Key-Value Pairs](#key-value-pairs)
3. [Array Lists](#array-lists)
4. [Hash Tables](#hash-tables)
5. [Hash Sets](#hash-sets)
6. [Stacks](#stacks)
7. [Queues](#queues)
8. [Immutable Array Lists](#immutable-array-lists)
9. [Immutable Hash Tables](#immutable-hash-tables)
10. [Immutable Hash Sets](#immutable-hash-sets)

<h2 id="introduction">Introduction</h2>

Unfortunately, PHP's support for collections is relatively incomplete.  The `array` type is reused for multiple types, like hash tables and lists.  PHP also has _some_ support for advanced types in the SPL library, but it is incomplete, and its syntax is somewhat clunky.  To cover for PHP's lack of coverage of collections, Opulence provides simple wrappers for common collections found in most other programming languages.

<h2 id="key-value-pairs">Key-Value Pairs</h2>

Like its name implies, `KeyValuePair` holds a key and a value.  An example usage of in Opulence's [`HashTable`](#hash-tables) and [`ImmutableHashTable` 
](#immutable-hash-tables).  To instantiate one, pass in the key and value:

```php
use Opulence\Collections\KeyValuePair;

$kvp = new KeyValuePair('thekey', 'thevalue');
```

<h4 id="key-value-pairs-getting-keys">KeyValuePair::getKey()</h4>

_Runtime: O(1)_

To get the key-value pair's key, call

```php
$kvp->getKey();
```

<h4 id="key-value-pairs-get">KeyValuePair::getValue()</h4>

_Runtime: O(1)_

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

<h4 id="array-lists-add">ArrayList::add()</h4>

_Runtime: O(1)_

You can add a value via

```php
$arrayList->add('foo');
```

<h4 id="array-lists-add-range">ArrayList::addRange()</h4>

_Runtime: O(n), n = numbers of values added_

You can add multiple values at once:

```php
$arrayList->addRange(['foo', 'bar']);
```

<h4 id="array-lists-clear">ArrayList::clear()</h4>

_Runtime: O(1)_

You can remove all values in the array list:

```php
$arrayList->clear();
```

<h4 id="array-lists-contains-value">ArrayList::containsValue()</h4>

_Runtime: O(n)_

To check for a value, call

```php
$containsValue = $arrayList->containsValue('foo');
```

<h4 id="array-lists-count">ArrayList::count()</h4>

_Runtime: O(1)_

To grab the number of values in the array list, call

```php
$count = $arrayList->count();
```

<h4 id="array-lists-get">ArrayList::get()</h4>

_Runtime: O(1)_

To get the value at a certain index from an array list, call

```php
$value = $arrayList->get(123);
```

If the index is out of range, an `OutOfRangeException` will be thrown.

<h4 id="array-lists-index-of">ArrayList::indexOf()</h4>

_Runtime: O(n)_

To grab the index for a value, call

```php
$index = $arrayList->indexOf('foo');
```

<h4 id="array-lists-insert">ArrayList::insert()</h4>

_Runtime: O(1)_

To insert a value at a specific index, call

```php
$arrayList->insert(23, 'foo');
```

<h4 id="array-lists-intersect">ArrayList::intersect()</h4>

_Runtime: O(nm)_

You can intersect an array list's values with an array by calling

```php
$arrayList->intersect(['foo', 'bar']);
```

If the array list doesn't contain the value, `null` will be returned.

<h4 id="array-lists-remove-value">ArrayList::removeIndex()</h4>

_Runtime: O(1)_

To remove a value by index, call

```php
$arrayList->removeIndex(123);
```

To remove a specific value, call

```php
$arrayList->removeValue('foo');
```

<h4 id="array-lists-reverse">ArrayList::reverse()</h4>

_Runtime: O(n)_

To reverse the values in the list, call

```php
$arrayList->reverse();
```

<h4 id="array-lists-sort">ArrayList::sort()</h4>

_Runtime: O(n log n)_

You can sort values similar to the way you can sort PHP arrays via `usort()`:

```php
$comparer = function ($a, $b) {
    return $a > $b ? 1 : -1;
};
$arrayList->sort($comparer);
```

<h4 id="array-lists-to-array">ArrayList::toArray()</h4>

_Runtime: O(1)_

You can get the underlying array by calling

```php
$array = $arrayList->toArray();
```

<h4 id="array-lists-union">ArrayList::union()</h4>

_Runtime: O(nm)_

You can union an array list's values with an array via

```php
$arrayList->union(['foo', 'bar']);
```

<h2 id="hash-tables">Hash Tables</h2>

Hash tables are most similar to PHP's built-in associative array functionality - they map keys to values.  Unlike PHP associative arrays (which only supports scalars as keys), Opulence's `HashTables` support scalars, objects, arrays, and resources as keys.  You can instantiate one with or without an array of key-value pairs:

```php
use Opulence\Collections\HashTable;

$hashTable = new HashTable();
// Or...
$hashTable = new HashTable([new KeyValuePair('foo', 'bar')]);
```

> **Note:** `HashTable` implements `ArrayAccess` and `IteratorAggregate`, so you can use array-like accessors and iterate over it. The keys will be numeric, and the values will be [key-value pairs](#key-values-pairs).

<h4 id="hash-tables-add">HashTable::add()</h4>

_Runtime: O(1)_

To add a value, call

```php
$hashTable->add('foo', 'bar');
```

<h4 id="hash-tables-add-range">HashTable::addRange()</h4>

_Runtime: O(n), n = number of values added_

To add multiple values at once, pass in an array of `KeyValuePair` objects:

```php
$kvps = [
    new KeyValuePair('foo', 'bar'),
    new KeyValuePair('baz', 'blah')
];
$hashTable->addRange($kvps);
```

<h4 id="hash-tables-clear">HashTable::clear()</h4>

_Runtime: O(1)_

You can remove all values:

```php
$hashTable->clear();
```

<h4 id="hash-tables-contains-key">HashTable::containsKey()</h4>

_Runtime: O(1)_

To check for a value, call

```php
$containsKey = $hashTable->containsKey('foo');
```

<h4 id="hash-tables-contains-value">HashTable::containsValue()</h4>

_Runtime: O(n)_

To check for a key, call

```php
$containsValue = $hashTable->containsValue('foo');
```

<h4 id="hash-tables-count">HashTable::count()</h4>

_Runtime: O(1)_

To get the number of values in the hash table, call

```php
$count = $hashTable->count();
```

<h4 id="hash-tables-get">HashTable::get()</h4>

_Runtime: O(1)_

To get a value at a key, call

```php
$value = $hashTable->get('foo');
```

If the value does not exist, an `OutOfBoundsException` will be thrown.

<h4 id="hash-tables-get-keys">HashTable::getKeys()</h4>

_Runtime: O(n)_

You can grab all of the keys in the hash table:

```php
$hashTable->getKeys();
```

<h4 id="hash-tables-get-values">HashTable::getValues()</h4>

_Runtime: O(n)_

You can grab all of the values in the hash table:

```php
$hashTable->getValues();
```

<h4 id="hash-tables-remove-key">HashTable::removeKey()</h4>

_Runtime: O(1)_

To remove a value at a certain key, call

```php
$hashTable->removeKey('foo');
```

<h4 id="hash-tables-remove-value">HashTable::removeValue()</h4>

_Runtime: O(n)_

To remove a value, call

```php
$hashTable->removeValue('foo');
```

<h4 id="hash-tables-to-array">HashTable::toArray()</h4>

_Runtime: O(n)_

To get the underlying array, call

```php
$array = $hashTable->toArray();
```

This will return a list of `KeyValuePair` - not an associative array.  The reason for this is that keys can be non-strings, which is not supported in PHP.

<h4 id="hash-tables-try-get">HashTable::tryGet()</h4>

_Runtime: O(1)_

If you would like to try to safely get a value that may or may not exist, use `tryGet()`.  It'll return `true` if the key exists, otherwise `false`.  It will also set the second parameter to the value if the key exists.

```php
$value = null;
$exists = $hashTable->tryGet('foo', $value);
```

<h2 id="hash-sets">Hash Sets</h2>

Hash sets are lists with unique values.  They accept objects, scalars, arrays, and resources as values.  You can instantiate one with or without an array of key => value pairs:

```php
use Opulence\Collections\HashSet;

$set = new HashSet();
// Or...
$set = new HashSet(['foo', 'bar']);
```

> **Note:** `HashSet` implements `ArrayAccess` and `IteratorAggregate`, so you can use array-like accessors and iterate over it.  When iterating, the keys will be numeric and the values will be the values in the set.

<h4 id="hash-sets-add">HashSet::add()</h4>

_Runtime: O(1)_

You can add a value via

```php
$set->add('foo');
```

<h4 id="hash-sets-add-range">HashSet::addRange()</h4>

_Runtime: O(n), n = number of values added_

You can add multiple values at once:

```php
$set->addRange(['foo', 'bar']);
```

<h4 id="hash-sets-clear">HashSet::clear()</h4>

_Runtime: O(1)_

To remove all values in the set, call `clear()`:

```php
$set->clear();
```

<h4 id="hash-sets-contains-value">HashSet::containsValue()</h4>

_Runtime: O(1)_

To check for a value, call

```php
$containsValue = $set->containsValue('foo');
```

<h4 id="hash-sets-count">HashSet::count()</h4>

_Runtime: O(1)_

To grab the number of values in the hash set, call

```php
$count = $set->count();
```

<h4 id="hash-sets-intersect">HashSet::intersect()</h4>

_Runtime: O(nm)_

You can intersect a hash set with an array by calling

```php
$set->intersect(['foo', 'bar']);
```

<h4 id="hash-sets-remove-value">HashSet::removeValue()</h4>

_Runtime: O(1)_

To remove a specific value, call

```php
$set->removeValue('foo');
```

<h4 id="hash-sets-sort">HashSet::sort()</h4>

_Runtime: O(n log n)_

You can sort values similar to the way you can sort PHP arrays via `usort()`:

```php
$comparer = function ($a, $b) {
    return $a > $b ? 1 : -1;
};
$set->sort($comparer);
```

<h4 id="hash-sets-to-array">HashSet::toArray()</h4>

_Runtime: O(n)_

To get the underlying array, call

```php
$array = $set->toArray();
```

<h4 id="hash-sets-union">HashSet::union()</h4>

_Runtime: O(nm)_

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

<h4 id="stacks-clear">Stack::clear()</h4>

_Runtime: O(1)_

To clear the values in the stack, call

```php
$stack->clear();
```

<h4 id="stacks-contains-value">Stack::containsValue()</h4>

_Runtime: O(n)_

To check for a value within a stack, call

```php
$containsValue = $stack->containsValue('foo');
```

<h4 id="stacks-count">Stack::count()</h4>

_Runtime: O(1)_

To get the number of values in the stack, call

```php
$count = $stack->count();
```

<h4 id="stacks-peek">Stack::peek()</h4>

_Runtime: O(1)_

To peek at the top value in the stack, call

```php
$value = $stack->peek();
```

<h4 id="stacks-pop">Stack::pop()</h4>

_Runtime: O(1)_

To pop a value off the stack, call

```php
$value = $stack->pop();
```

If there are no values in the stack, this will return `null`.

<h4 id="stacks-push">Stack::push()</h4>

_Runtime: O(1)_

To push a value onto the stack, call

```php
$stack->push('foo');
```

If there are no values in the stack, this will return `null`.

<h4 id="stacks-to-array">Stack::toArray()</h4>

_Runtime: O(1)_

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

<h4 id="queues-clear">Queue::clear()</h4>

_Runtime: O(1)_

To clear the queue, call

```php
$queue->clear();
```

<h4 id="queues-contains-value">Queue::containsValue()</h4>

_Runtime: O(n)_

To check for a value within a queue, call

```php
$containsValue = $queue->containsValue('foo');
```

<h4 id="queues-count">Queue::count()</h4>

_Runtime: O(1)_

To get the number of values in the queue, call

```php
$count = $queue->count();
```

<h4 id="queues-dequeue">Queue::dequeue()</h4>

_Runtime: O(1)_

To dequeue a value from the queue, call

```php
$value = $queue->dequeue();
```

If there are no values in the queue, this will return `null`.

<h4 id="queues-enqueue">Queue::enqueue()</h4>

_Runtime: O(1)_

To enqueue a value onto the queue, call

```php
$queue->enqueue('foo');
```

<h4 id="queues-peek">Queue::peek()</h4>

_Runtime: O(1)_

To peek at the value at the beginning of the queue, call

```php
$value = $queue->peek();
```

If there are no values in the queue, this will return `null`.

<h4 id="queues-to-array">Queue::toArray()</h4>

_Runtime: O(1)_

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

<h4 id="immutable-array-lists-contains-value">ImmutableArrayList::containsValue()</h4>

_Runtime: O(n)_

To check for a value, call

```php
$containsValue = $arrayList->containsValue('foo');
```

<h4 id="immutable-array-lists-count">ImmutableArrayList::count()</h4>

_Runtime: O(1)_

To grab the number of values in the array list, call

```php
$count = $arrayList->count();
```

<h4 id="immutable-array-lists-get">ImmutableArrayList::get()</h4>

_Runtime: O(1)_

To get the value at a certain index from an array list, call

```php
$value = $arrayList->get(123);
```

If the index is out of range, an `OutOfRangeException` will be thrown.

<h4 id="immutable-array-lists-index-of">ImmutableArrayList::indexOf()</h4>

_Runtime: O(n)_

To grab the index for a value, call

```php
$index = $arrayList->indexOf('foo');
```

If the array list doesn't contain the value, `null` will be returned.

<h4 id="immutable-array-lists-to-array">ImmutableArrayList::toArray()</h4>

_Runtime: O(1)_

If you want to grab the underlying array, call

```php
$array = $arrayList->toArray();
```

<h2 id="immutable-hash-tables">Immutable Hash Tables</h2>

Sometimes, your business logic might dictate that a [hash table](#hash-tables) is read-only.  Opulence provides support via `ImmutableHashTable`.  It requires that you pass key-value pairs into its constructor:

```php
use Opulence\Collections\ImmutableHashTable;

$hashTable = new ImmutableHashTable([new KeyValuePair('foo', 'bar')]);
```

> **Note:** `ImmutableHashTable` implements `ArrayAccess` and `IteratorAggregate`, so you can use array-like accessors and iterate over it. When iterating, the keys will be numeric, and the values will be [key-value pairs](#key-values-pairs).

<h4 id="immutable-hash-tables-contains-key">ImmutableHashTable::containsKey()</h4>

_Runtime: O(1)_

To check for a key, call

```php
$containsKey = $hashTable->containsKey('foo');
```

<h4 id="immutable-hash-tables-contains-value">ImmutableHashTable::containsValue()</h4>

_Runtime: O(n)_

To check for a value, call

```php
$containsValue = $hashTable->containsValue('foo');
```

<h4 id="immutable-hash-tables-count">ImmutableHashTable::count()</h4>

_Runtime: O(1)_

To get the number of values in the hash table, call

```php
$count = $hashTable->count();
```

<h4 id="immutable-hash-tables-get">ImmutableHashTable::get()</h4>

_Runtime: O(1)_

To get a value at a key, call

```php
$value = $hashTable->get('foo');
```

If the value does not exist, an `OutOfBoundsException` will be thrown.

<h4 id="immutable-hash-tables-get-keys">ImmutableHashTable::getKeys()</h4>

_Runtime: O(n)_

You can grab all of the keys in the hash table:

```php
$hashTable->getKeys();
```

<h4 id="immutable-hash-tables-get-values">ImmutableHashTable::getValues()</h4>

_Runtime: O(n)_

You can grab all of the values in the hash table:

```php
$hashTable->getValues();
```

<h4 id="immutable-hash-tables-to-array">ImmutableHashTable::toArray()</h4>

_Runtime: O(n)_

To get the underlying array, call

```php
$array = $hashTable->toArray();
```

This will return a list of `KeyValuePair` - not an associative array.  The reason for this is that keys can be non-strings (eg objects) in hash tables, but keys in PHP associative arrays must be serializable.

<h4 id="immutable-hash-tables-try-get">ImmutableHashTable::tryGet()</h4>

_Runtime: O(1)_

If you would like to try to safely get a value that may or may not exist, use `tryGet()`.  It'll return `true` if the key exists, otherwise `false`.  It will also set the second parameter to the value if the key exists.

```php
$value = null;
$exists = $hashTable->tryGet('foo', $value);
```

<h2 id="immutable-hash-sets">Immutable Hash Sets</h2>

Immutable hash sets are read-only [hash sets](#hash-sets).  They accept objects, scalars, arrays, and resources as values.  You can instantiate one with a list of values:

```php
use Opulence\Collections\ImmutableHashSet;

$set = new ImmutableHashSet(['foo', 'bar']);
```

> **Note:** `ImmutableHashSet` implements `ArrayAccess` and `IteratorAggregate`, so you can use array-like accessors and iterate over it.

<h4 id="immutable-hash-sets-contains-value">ImmutableHashSet::containsValue()</h4>

_Runtime: O(1)_

To check for a value, call

```php
$containsValue = $set->containsValue('foo');
```

<h4 id="immutable-hash-sets-count">ImmutableHashSet::count()</h4>

_Runtime: O(1)_

To grab the number of values in the set, call

```php
$count = $set->count();
```

<h4 id="immutable-hash-sets-to-array">ImmutableHashSet::toArray()</h4>

_Runtime: O(n)_

To get the underlying array, call

```php
$array = $set->toArray();
```