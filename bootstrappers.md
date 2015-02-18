# Bootstrappers

## Table of Contents
1. [Introduction](#introduction)
2. [Registering Bindings](#registering-bindings)
3. [Running Bootstrappers](#running-bootstrappers)
4. [Bootstrapper Example](#bootstrapper-example)

<h2 id="introduction">Introduction</h2>
Most applications need to do some configuration before starting.  A common task is registering bindings, and yet another is setting up database connections.  You can do this bootstrapping by extending `RDev\Applications\Bootstrappers\Bootstrapper`.  It accepts `Paths`, `Environment`, and `ISession` objects in its constructor, which can be useful for something like binding a particular database instance based on the current environment.

<h2 id="registering-bindings">Registering Bindings</h2>
Before you can start using your application, your IoC container needs some bindings to be registered.  This is where `Bootstrapper::registerBindings()` comes in handy.  Anything that needs to be bound to the IoC container should be done here.  Once the application is started, all bootstrappers' bindings are registered.

<h2 id="running-bootstrappers">Running Bootstrappers</h2>
Bootstrappers also support a `run()` command, which is run AFTER all bindings have been registered.  This is useful for any last configuration that needs to be performed before the application is run.  Use type-hinted parameters in `run()` for any dependencies your bootstrapper depends on to run successfully.

<h2 id="bootstrapper-example">Bootstrapper Example</h2>
Let's pretend you're developing an application that grabs WordPress posts from a database and displays them in a nice view.  You might have a `Posts` class, which needs a database connection to read the posts.  Let's take a look at a simple bootstrapper:

```php
namespace MyApp\WordPress\Bootstrappers;
use MyApp\WordPress;
use RDev\Applications\Bootstrappers;
use RDev\Databases\SQL;
use RDev\IoC;

class MyBootstrapper extends Bootstrappers\Bootstrapper
{
    // This will be run before any bootstrappers' run() methods have been called
    public function registerBindings(IoC\IContainer $container)
    {
        // Create our Posts object
        $container->bind("MyApp\\WordPress\\Posts", new WordPress\Posts());
    }

    // The IoC container will automatically resolve these dependencies
    public function run(WordPress\Posts $posts, SQL\ConnectionPool $connectionPool)
    {
        // Bind the connection pool to our posts object
        $posts->setDatabaseConnection($connectionPool->getReadConnection());
    }
}
```

You can now inject `WordPress\Posts` into any service or controller that needs to query WordPress posts.