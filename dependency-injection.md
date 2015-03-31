# Dependency Injection

## Table of Contents
1. [Introduction](#introduction)
  1. [Explanation of Dependency Injection](#explanation-of-dependency-injection)
  2. [Dependency Injection Container](#dependency-injection-container)
2. [Basic Usage](#basic-usage)
3. [Binding a Specific Instance](#binding-a-specific-instance)
4. [Targeted Bindings](#targeted-bindings)
5. [Creating New Instances](#creating-new-instances)
6. [Creating Shared Instances](#creating-shared-instances)
7. [Passing Constructor Primitives](#passing-constructor-primitives)
8. [Using Setters](#using-setters)
9. [Calling Methods](#calling-methods)
10. [Getting a Binding](#getting-a-binding)
11. [Removing a Binding](#removing-a-binding)

<h2 id="introduction">Introduction</h2>
<h4 id="explanation-of-dependency-injection">Explanation of Dependency Injection</h4>
*Dependency Injection* refers to the practice of passing a class its dependencies instead of the class creating them on its own.  This is very useful for creating loosely-coupled, testable code.  Let's take a look at an example that doesn't use dependency injection:

```php
class Foo
{
    private $database;

    public function __construct()
    {
        $this->database = new Database();
    }

    public function insertIntoDatabase($query)
    {
        return $this->database->insert($query);
    }
}
```

Databases are complex, and unit testing them is very tricky.  To make unit testing simpler, we could mock the database class so that we don't ever actually query a real database:
```php
class DatabaseMock extends Database
{
    public function insert($query)
    {
        return true;
    }
}
```

The issue with `Foo` is that it creates its own instance of `Database`, so there's no way to pass it `DatabaseMock` without having to rewrite the class just for the test.  The solution is to "inject" the `Database` dependency into `Foo`:
```php
class Foo
{
    private $database;

    public function __construct(Database $database)
    {
        $this->database = $database;
    }

    public function insertIntoDatabase($query)
    {
        return $this->database->insert($query);
    }
}
```

The difference is subtle, but now we can easily inject `DatabaseMock` when writing unit tests:
```php
$database = new DatabaseMock();
$foo = new Foo($database);
echo $foo->insertIntoDatabase("bar"); // 1
```

By inverting the control of dependencies (meaning classes no longer maintain their own dependencies), we've made our code easier to test.

<h4 id="dependency-injection-container">Dependency Injection Container</h4>
Hopefully, you can see that injecting dependencies is a simple, yet powerful feature.  Now the question is "Where should I inject the dependencies from?"  The answer is a **dependency injection container** (we'll call it a **container** from here on out).  A container can take a look at a constructor/setter methods and determine what dependencies a class relies on.  It creates a collection of various dependencies and automatically injects them into classes.  One of the coolest features of containers is the ability to bind a concrete class to an interface or abstract class.  In other words, it'll inject the concrete class implementation whenever there's a dependency on its interface or base class.  This frees you to "code to an interface, not an implementation".  At runtime, you can bind classes to interfaces, and execute your code.

<h2 id="basic-usage">Basic Usage</h2>
The **container** looks at type hints in methods to determine the type of dependency a class relies on.  The container even lets you specify values for primitive types, eg strings and numbers.

Let's take a look at a class `A` that has a dependency on `IFoo`:
```php
interface IFoo
{
    public function sayHi();
}

class ConcreteFoo implements IFoo
{
    public function sayHi()
    {
        echo "Hi";
    }
}

class A
{
    private $foo;

    public function __construct(IFoo $foo)
    {
        $this->foo = $foo;
    }

    public function getFoo()
    {
        return $this->foo;
    }
}
```

If we always want to pass in an instance of `ConcreteFoo` when there's a dependency on `IFoo`, we can bind the two:
```php
use RDev\IoC\Container;

$container = new Container();
$container->bind("IFoo", "ConcreteFoo");
```

Now, whenever a dependency on `IFoo` is detected, the container will inject an instance of `ConcreteFoo`.  To create an instance of `A` with its dependencies set, simply:
```php
$a = $container->makeNew("A");
$a->getFoo()->sayHi(); // "Hi"
```

As you can see, the container automatically injected an instance of `ConcreteFoo`.

<h2 id="binding-a-specific-instance">Binding a Specific Instance</h2>
Binding a specific instance to an interface is also possible through the `bind()` method:
```php
$concreteInstance = new ConcreteFoo();
$container->bind("IFoo", $concreteInstance);
echo $concreteInstance === $container->makeShared("IFoo"); // 1
```

<h2 id="targeted-bindings">Targeted Bindings</h2>
By default, bindings are registered so that they can be used by all classes.  If you'd like to bind a concrete class to an interface or abstract class for only a specific class, you can create a targeted binding:
```php
$container->bind("IFoo", "ConcreteFoo", "A");
```

Now, `ConcreteFoo` is only bound to `IFoo` for the target class `A`.

> **Note:** Targeted bindings take precedence over universal bindings.

<h2 id="creating-new-instances">Creating New Instances</h2>
To create a brand new instance of a class with all of its dependencies injected, you can call `makeNew()`:
```php
$container->bind("IFoo", "ConcreteFoo");
$a1 = $container->makeNew("A");
$a2 = $container->makeNew("A");
echo $a1 === $a2; // 0
```

<h2 id="creating-shared-instances">Creating Shared Instances</h2>
Shared instances are just that - shared.  No matter how many times you make a shared instance, you'll always get the same instance.  This concept is similar to the **Singleton** design pattern, but with the added benefit of being able to bind different concrete implementations at runtime.  To create a shared instance of a class with all of its dependencies injected, you can call `makeShared()`:
```php
$container->bind("IFoo", "ConcreteFoo");
$a1 = $container->makeShared("A");
$a2 = $container->makeShared("A");
echo $a1 === $a2; // 1
```

<h2 id="passing-constructor-primitives">Passing Constructor Primitives</h2>
If your constructor depends on some primitive values, you can set them in both the `makeNew()` and `makeShared()` methods:
```php
class B
{
    private $foo;
    private $additionalMessage;

    public function __construct(IFoo $foo, $additionalMessage)
    {
        $this->foo = $foo;
        $this->additionalMessage = $additionalMessage;
    }

    public function getFoo()
    {
        return $this->foo;
    }

    public function sayAdditionalMessage()
    {
        echo $this->additionalMessage;
    }
}

$container->bind("IFoo", "ConcreteFoo");
$b = $container->makeNew("B", ["I love containers!"]);
echo get_class($b->getFoo()); // "ConcreteFoo"
$b->sayAdditionalMessage(); // "I love containers!"
```

Only the primitive values should be passed in the array.  They must appear in the same order as the constructor.

<h2 id="using-setters">Using Setters</h2>
Sometimes a class needs setter methods to pass in dependencies.  This is possible using both the `makeNew()` and `makeShared()` methods:
```php
class C
{
    private $foo;
    private $additionalMessage;

    public function __construct()
    {
        // Don't do anything
    }

    public function getFoo()
    {
        return $this->foo;
    }

    public function sayAdditionalMessage()
    {
        echo $this->additionalMessage;
    }

    public function setFoo(IFoo $foo)
    {
        $this->foo = $foo;
    }

    public function setFooAndAdditionalMessage(IFoo $foo, $additionalMessage)
    {
        $this->foo = $foo;
        $this->additionalMessage = $additionalMessage;
    }
}

$container->bind("IFoo", "ConcreteFoo");
$c = $container->makeNew("C", [], ["setFoo" => []]);
echo get_class($c->getFoo()); // "ConcreteFoo"
```

If your setter requires primitive values, you can pass them in, too:
```php
$container->bind("IFoo", "ConcreteFoo");
$c = $container->makeNew("C", [], ["setFooAndAdditionalMessage" => ["I love setters!"]]);
echo get_class($c->getFoo()); // "ConcreteFoo"
$c->sayAdditionalMessage(); // "I love setters!"
```

<h2 id="calling-methods">Calling Methods</h2>
It's possible to call methods on a class using the container to resolve dependencies using `call()`:

```php
class D
{
    private $foo;
    private $bar;

    public function getBar()
    {
        return $this->bar;
    }

    public function getFoo()
    {
        return $this->foo;
    }

    public function setFoo(IFoo $foo, $bar)
    {
        $this->foo = $foo;
        $this->bar = $bar;
    }
}

$container->bind("IFoo", "ConcreteFoo");
$instance = new D();
$container->call($instance, "setFoo", ["Primitive was set"]);
echo get_class($c->getFoo()); // "ConcreteFoo"
echo $instance->getBar(); // "Primitive was set"
```

<h2 id="getting-a-binding">Getting a Binding</h2>
To get the current binding for an interface, call `getBinding()`.  To check whether or not a binding exists, call `isBound()`.
```php
$container->bind("IFoo", "ConcreteFoo");
echo $container->getBinding("IFoo"); // "ConcreteFoo"
echo $container->isBound("IFoo"); // 1
// Non-existent bindings return null
echo $container->getBinding("NonExistentInterface"); // null
echo $container->isBound("NonExistentInterface"); // 0
```

Similarly, you can get a current targeted binding:
```php
$container->bind("IFoo", "ConcreteFoo", "A");
echo $container->getBinding("IFoo", "A"); // "ConcreteFoo"
echo $container->isBound("IFoo", "A"); // 1
// Non-existent targeted bindings return null
echo $container->getBinding("NonExistentInterface", "A"); // null
echo $container->isBound("NonExistentInterface", "A"); // 0
```

> **Note:** If a target is specified, but nothing has been explicitly bound to it, then `getBinding()` returns any universal bindings, and `isBound()` returns false.  Therefore, checking if something is bound to a target using the result from `getBinding()` could be misleading.

<h2 id="removing-a-bindings">Removing a Binding</h2>
To remove a binding, call `unbind()`:
```php
$container->bind("IFoo", "ConcreteFoo");
$container->unbind("IFoo");
echo $container->isBound("IFoo"); // 0
echo $container->getBinding("IFoo"); // null
```

To remove a targeted binding:
```php
$container->bind("IFoo", "ConcreteFoo", "A");
$container->unbind("IFoo", "A");
echo $container->isBound("IFoo", "A"); // 0
echo $container->getBinding("IFoo", "A"); // null
```