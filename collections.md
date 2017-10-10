# Collections

## Table of Contents
1. [Introduction](#introduction)
2. [Array Lists](#array-lists)
    1. [Adding Values](#array-lists-adding-values)
    2. [Checking for Values](#array-lists-checking-for-values)
    3. [Getting Values](#array-lists-getting-values)
    4. [Getting Indices](#array-lists-getting-indices)
    5. [Getting the Count](#array-lists-getting-count)
    6. [Removing Values](#array-lists-removing-values)
    7. [Reversing the Values](#array-lists-reversing-values)
    8. [Sorting Values](#array-lists-sorting-values)
3. [Hash Tables](#hash-tables)
    1. [Adding Values](#hash-tables-adding-values)
    2. [Getting Values](#hash-tables-getting-values)
    3.  [Checking for Values](#hash-tables-checking-for-values)
    4. [Getting the Count](#hash-tables-getting-count)
    5. [Removing Values](#hash-tables-removing-values)
4. [Read-Only Hash Tables](#read-only-hash-tables)
    1. [Getting Values](#read-only-hash-tables-getting-values)
    2. [Checking for Values](#read-only-hash-tables-checking-for-values)
    3. [Getting the Count](#read-only-hash-tables-getting-count)
5. [Stacks](#stacks)
    1. [Pushing Values](#stacks-pushing-values)
    2. [Popping Values](#stacks-popping-values)
    3. [Peeking at Values](#stacks-peeking-at-values)
    4. [Checking for Values](#stacks-checking-for-values)
    5. [Getting the Count](#stacks-getting-count)
    6. [Clearing Values](#stacks-clearing-values)
6. [Queues](#queues)
    1. [Enqueueing Values](#queues-enqueueing-values)
    2. [Dequeueing Values](#queues-dequeueing-values)
    3. [Peeking at Values](#queues-peeking-at-values)
    4. [Checking for Values](#queues-checking-for-values)
    5. [Getting the Count](#queues-getting-count)
    6. [Clearing Values](#queues-clearing-values)

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

> **Note:** `ArrayList` implements `ArrayAccess` and `IteratorAggregate`, so you can use array-like accessors, and you can iterate over it.

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

<h4 id="array-lists-checking-for-values">Checking for Values</h4>

To check for a value, call

```php
$containsValue = $arrayList->containsValue('foo');
```

<h4 id="array-lists-getting-values">Getting Values</h4>

To get the value at a certain index from an array list, call

```php
$value = $arrayList->get(123);
```

<h4 id="array-lists-getting-indices">Getting Indices</h4>

To grab the index for a value, call

```php
$index = $arrayList->indexOf('foo');
```

If the array list doesn't contain the value, `null` will be returned.

<h4 id="array-lists-getting-count">Getting the Count</h4>

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

> **Note:** `HashTable` implements `ArrayAccess` and `IteratorAggregate`, so you can iterate over it.

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

<h4 id="hash-tables-getting-count">Getting the Count</h4>

To get the number of values in the hash table, call

```php
$count = $hashTable->count();
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

<h4 id="hash-tables-removing-values">Removing Values</h4>

To remove a value at a certain key, call

```php
$hashTable->removeKey('foo');
```

<h2 id="read-only-hash-tables">Read-Only Hash Tables</h2>

Sometimes, your business logic might dictate that a hash table is read-only.  Opulence provides support via `ReadOnlyHashTable`.  It requires that you pass values into its constructor:

```php
use Opulence\Collections\ReadOnlyHashTable;

$hashTable = new ReadOnlyHashTable(['foo' => 'bar', 'baz' => 'blah']);
```

To get the underlying array, call

```php
$array = $hashTable->toArray();
```

> **Note:** `ReadOnlyHashTable` implements `IteratorAggregate`, so you can iterate over it.

<h4 id="read-only-hash-tables-getting-values">Getting Values</h4>

To get a value at a key, call

```php
$value = $hashTable->get('foo');
```

<h4 id="read-only-hash-tables-checking-for-values">Checking for Values</h4>

To check for a key, call

```php
$containsKey = $hashTable->containsKey('foo');
```

To check for a value, call

```php
$containsValue = $hashTable->containsValue('foo');
```

<h4 id="read-only-hash-tables-getting-count">Getting the Count</h4>

To get the number of values in the hash table, call

```php
$count = $hashTable->count();
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

<h4 id="stacks-getting-count">Getting the Count</h4>

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

<h4 id="queues-getting-count">Getting the Count</h4>

To get the number of values in the queue, call

```php
$count = $queue->count();
```

<h4 id="queues-clearing-values">Clearing Values</h4>

To clear the queue, call

```php
$queue->clear();
```