# IoC Container

## Table of Contents
1. [Introduction](#introduction)
  1. [Explanation of Dependency Injection](#explanation-of-dependency-injection)
  2. [Dependency Injection Container](#dependency-injection-container)
2. [Basic Usage](#basic-usage)
3. [Binding Instances](#binding-instances)
4. [Binding Singletons](#binding-singletons)
5. [Binding Prototypes](#binding-prototypes)
6. [Binding Factories](#binding-factories)
7. [Targeting](#targeting)
8. [Bootstrappers](#bootstrappers)
9. [Calling Methods](#calling-methods)
10. [Calling Closures](#calling-closures)
11. [Checking a Binding](#checking-a-binding)
12. [Removing a Binding](#removing-a-binding)
13. [Applications in Opulence](#applications-in-opulence)

<h2 id="introduction">Introduction</h2>
<h4 id="explanation-of-dependency-injection">Explanation of Dependency Injection</h4>
**Dependency Injection** refers to the practice of passing a class its dependencies instead of the class creating them on its own.  This is very useful for creating loosely-coupled, testable code.  Let's take a look at an example that doesn't use dependency injection:

```php
class Foo
{
    private $database;

    public function __construct()
    {
        $this->database = new Database();
    }

    public function insertIntoDatabase(string $query) : bool
    {
        return $this->database->insert($query);
    }
}
```

Databases are complex, and unit testing them is very tricky.  To make unit testing simpler, we could mock the database class so that we don't ever actually query a real database:
```php
class DatabaseMock extends Database
{
    public function insert(string $query) : bool
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

    public function insertIntoDatabase($query) : bool
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

> **Note:**  Classes that accept only concrete classes in their constructors do not need to be bound to the container; they can be instantiated automatically.  A class should only be bound to the container if it depends on an interface, abstract class, or primitive.

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

    public function getFoo() : IFoo
    {
        return $this->foo;
    }
}
```

If we always want to pass in an instance of `ConcreteFoo` when there's a dependency on `IFoo`, we can bind the two:
```php
use Opulence\Ioc\Container;

$container = new Container();
$container->bindSingleton("IFoo", "ConcreteFoo");
```

Now, whenever a dependency on `IFoo` is detected, the container will inject an instance of `ConcreteFoo`.  To create an instance of `A` with its dependencies set, simply:
```php
$a = $container->resolve("A");
$a->getFoo()->sayHi(); // "Hi"
```

As you can see, the container automatically injected an instance of `ConcreteFoo`.  You can also bind a value to multiple interfaces with a single call:

```php
$concreteFoo = new ConcreteFoo();
// $concreteFoo will be bound to both "IFoo" and "ConcreteFoo"
$container->bindInstance(["IFoo", "ConcreteFoo"], $concreteFoo);
```

<h2 id="binding-instances">Binding Instances</h2>
Binding a specific instance to an interface is also possible through the `bindInstance()` method.  Every time you resolve the interface, this instance will be returned.
```php
$concreteInstance = new ConcreteFoo();
$container->bindInstance("IFoo", $concreteInstance);
echo $concreteInstance === $container->resolve("IFoo"); // 1
```

<h2 id="binding-singletons">Binding Singletons</h2>
You can bind an interface to a class name and have it always resolve to the same instance of the class (also known as a singleton).
```php
$container->bindSingleton("IFoo", "ConcreteFoo");
echo get_class($container->resolve("IFoo")); // "ConcreteFoo"
echo $container->resolve("IFoo") === $container->resolve("IFoo"); // 1
```

If your concrete class requires any primitive values, pass them in an array in the same order they appear in the constructor.

<h2 id="binding-prototypes">Binding Prototypes</h2>
You can bind an interface to a class name and have it always resolve to a new instance of the class (also known as a prototype).
```php
$container->bindPrototype("IFoo", "ConcreteFoo");
echo get_class($container->resolve("IFoo")); // "ConcreteFoo"
echo $container->resolve("IFoo") === $container->resolve("IFoo"); // 0
```

If your concrete class requires any primitive values, pass them in an array in the same order they appear in the constructor.

<h2 id="binding-factories">Binding Factories</h2>
You can bind any `callable` to act as a factory to resolve an interface.  Factories are only evaluated when they're needed.

```php
$container->bindFactory("IFoo", function () {
    return new ConcreteFoo();
});
echo get_class($container->resolve("IFoo")); // "ConcreteFoo" 
```

> **Note:** Factories must be parameterless.

By default, resolving interfaces that were bound with a factory will return a new instance each time you call `resolve()`.  If you'd like the instance created by the factory to be bound as a singleton, specify `true` as the last parameter:

```php
$container->bindFactory("IFoo", function () {
    return new ConcreteFoo();
}, true);
echo $container->resolve("IFoo") === $container->resolve("IFoo"); // 1
```

<h2 id="targeting">Targeting</h2>
By default, bindings are registered so that they can be used by all classes.  If you'd like to bind a concrete class to an interface or abstract class for only a specific class, you can create a targeted binding using `for(TARGET_CLASS_NAME)` before your binding method:
```php
$container->for("A", function ($container) {
    $container->bindSingleton("IFoo", "ConcreteFoo");
});
```

Now, `ConcreteFoo` is only bound to `IFoo` for the target class `A`.

> **Note:** Targeted bindings take precedence over universal bindings.

Targeting works for the following methods:

* `bindFactory()`
  * Binds a factory for a target
* `bindInstance()`
  * Binds an instance for a target
* `bindPrototype()`
  * Binds a prototype for a target
* `bindSingleton()`
  * Binds a singleton for a target
* `hasBinding()`
  * Checks if a target has a binding for the input interface
* `resolve()`
  * Resolves an interface by first checking for targeted bindings, and then universal bindings
* `unbind()`
  * Unbinds the interface from the target
  
<h2 id="bootstrappers">Bootstrappers</h2>
Sometimes, you'll find yourself needing to bind several components of your module to your IoC container.  To keep yourself from writing repetitive code to do these bindings, you can use [bootstrappers](bootstrappers).  They're perfect for plugging-and-playing whole modules into your application.

To learn more about them, read [their docs](bootstrappers).

<h2 id="calling-methods">Calling Methods</h2>
It's possible to call methods on a class using the container to resolve dependencies using `callMethod()`:

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

$container->bindSingleton("IFoo", "ConcreteFoo");
$instance = new D();
$container->callMethod($instance, "setFoo", ["Primitive was set"]);
echo get_class($c->getFoo()); // "ConcreteFoo"
echo $instance->getBar(); // "Primitive was set"
```

<h2 id="calling-closures">Calling Closures</h2>
You can use `callClosure()` to automatically inject parameters into any closure:

```php
echo $container->callClosure(
    function (Foo $foo, $somePrimitive) {
        return get_class($foo) . ":" . $somePrimitive;
    },
    ["123"] // Pass in any primitive values
);
```

This will output:
```
Foo:123
```

<h2 id="checking-a-binding">Checking a Binding</h2>
To check whether or not a binding exists, call `hasBinding()`.
```php
$container->bindSingleton("IFoo", "ConcreteFoo");
echo $container->hasBinding("IFoo"); // 1
echo $container->hasBinding("NonExistentInterface"); // 0
```

<h2 id="removing-a-binding">Removing a Binding</h2>
To remove a binding, call `unbind()`:
```php
$container->bindSingleton("IFoo", "ConcreteFoo");
$container->unbind("IFoo");
echo $container->hasBinding("IFoo"); // 0
```

<h2 id="applications-in-opulence">Applications in Opulence</h2>
If you use Opulence's [command library](console-basics), the container automatically resolves the `Command` class.  Also, the [routing library](routing) uses `Opulence\Routing\Dispatchers\IDependencyResolver` to automatically resolve a matched controller.  Typically, Opulence's container is used by `IDependencyResolver` to do the resolution.