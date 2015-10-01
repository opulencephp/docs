# Bootstrappers

## Table of Contents
1. [Introduction](#introduction)
2. [Registering Bindings](#registering-bindings)
3. [Running Bootstrappers](#running-bootstrappers)
4. [Shutting Down Bootstrappers](#bootstrapper-shutdown)
5. [Bootstrapper Example](#bootstrapper-example)
6. [Lazy Bootstrappers](#lazy-bootstrappers)
  1. [Example](#lazy-example)
  2. [Caching](#bootstrapper-caching)

<h2 id="introduction">Introduction</h2>
Most applications need to do some configuration before starting.  For example, they might need to setup a database connection, configure which view engine to use, or assign the authentication scheme to use.  Because Opulence uses [dependency injection](dependency-injection) heavily, it's important that you set your bindings in the IoC container.  Bootstrappers are the place to do this.  They're loaded before the request is handled.  Typically, they register bindings to the IoC container, but they can also perform any necessary logic once all of the bindings are registered or before the application is shut down.  Bootstrappers extend `Opulence\Applications\Bootstrappers\Bootstrapper`.  They accept `Paths` and `Environment` objects in their constructors, which can be useful for something like binding a particular database instance based on the current environment.

<h2 id="registering-bindings">Registering Bindings</h2>
Before you can start using your application, your IoC container needs some bindings to be registered.  This is where `Bootstrapper::registerBindings()` comes in handy.  Anything that needs to be bound to the IoC container should be done here.  Once the application is started, all bootstrappers' bindings are registered.

<h2 id="running-bootstrappers">Running Bootstrappers</h2>
Bootstrappers also support a `run()` command, which is run AFTER all bindings have been registered.  This is useful for any last configuration that needs to be performed before the application is run.  Use type-hinted parameters in `run()` for any dependencies your bootstrapper depends on to run successfully.

<h2 id="bootstrapper-shutdown">Shutting Down Bootstrappers</h2>
Bootstrappers also support a `shutdown()` command, which is run as a pre-shutdown task on the application.  This is useful for any cleaning up a bootstrapper needs to do, such as writing session data.  Use type-hinted parameters in `shutdown()` for any dependencies your bootstrapper depends on to shutdown successfully.

<h2 id="bootstrapper-example">Bootstrapper Example</h2>
Let's pretend you're developing an application that grabs WordPress posts from a database and displays them in a nice view.  You might have a `Posts` class, which needs a database connection to read the posts.  Let's take a look at a simple bootstrapper:

```php
namespace MyApp\Bootstrappers\WordPress;

use MyApp\WordPress\Posts;
use Opulence\Applications\Bootstrappers\Bootstrapper;
use Opulence\Databases\ConnectionPool;
use Opulence\IoC\IContainer;

class MyBootstrapper extends Bootstrapper
{
    // This will be run before any bootstrappers' run() methods have been called
    public function registerBindings(IContainer $container)
    {
        // Create our Posts object
        $container->bind(Posts::class, new Posts());
    }

    // The IoC container will automatically resolve these dependencies
    public function run(Posts $posts, ConnectionPool $connectionPool)
    {
        // Bind the connection pool to our posts object
        $posts->setDatabaseConnection($connectionPool->getReadConnection());
    }
}
```

You can now inject `Posts` into any service or controller that needs to query WordPress posts.

<h2 id="lazy-bootstrappers">Lazy Bootstrappers</h2>
It's not very efficient to create, register bindings, run, and shut down every bootstrapper in your application when they're not all needed.  Sometimes, you may only like a bootstrapper to be registered/run/shut down if its bindings are required.  This is the purpose of **lazy bootstrappers**.  In Opulence, you can designate a bootstrapper to be lazy-loaded by making it implement `Opulence\Applications\Bootstrappers\ILazyBootstrapper`, which requires a `getBindings()` method to be defined.  This method should return a list of all classes/interfaces bound to the IoC container by that bootstrapper.  Let's take a look at an example:

<h4 id="lazy-example">Example</h4>
```php
namespace MyApp\Bootstrappers;

use MyApp\ConcreteFoo;
use MyApp\IFoo;
use Opulence\Applications\Bootstrappers\Bootstrapper;
use Opulence\Applications\Bootstrappers\ILazyBootstrapper;
use Opulence\IoC\IContainer;

class MyBootstrapper extends Bootstrapper implements ILazyBootstrapper
{
    public function getBindings()
    {
        return [IFoo::class];
    }
    
    public function registerBindings(IContainer $container)
    {
        $container->bind(IFoo::class, ConcreteFoo::class);
    }
}
```

<h4 id="bootstrapper-caching">Caching</h4>
Opulence automatically caches data about its lazy and eager (ie not lazy) bootstrappers.  This way, it doesn't have to instantiate each bootstrapper to determine which kind it is.  It also remembers which classes are bound by which bootstrappers.  If you add/remove/modify any bootstrappers, you must run [`php opulence framework:flushcache`](console-basics#frameworkflushcache) command in the console to flush this cache.