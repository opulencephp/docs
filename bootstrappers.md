# Bootstrappers

## Table of Contents
1. [Introduction](#introduction)
2. [Registering Bindings](#registering-bindings)
3. [Running Bootstrappers](#running-bootstrappers)
4. [Shutting Down Bootstrappers](#bootstrapper-shutdown)
5. [Lazy Bootstrappers](#lazy-bootstrappers)
  1. [Targeted Bindings](#targeted-bindings)
  2. [Caching](#bootstrapper-caching)
6. [Environment Variables](#environment-variables)
7. [Config Values](#config-values)

<h2 id="introduction">Introduction</h2>
Most applications need to do some configuration before starting.  For example, they might need to setup a database connection, configure which view engine to use, or assign the authentication scheme to use.  Because Opulence uses [dependency injection](dependency-injection) heavily, it's important that you set your bindings in the IoC container.  Bootstrappers are the place to do this.
  
Bootstrappers are loaded before the request is handled.  Typically, they register bindings to the IoC container, but they can also perform any necessary logic once all of the bindings are registered or before the application is shut down.  Bootstrappers extend `Opulence\Ioc\Bootstrappers\Bootstrapper`.  Their constructors are empty.

<h2 id="registering-bindings">Registering Bindings</h2>
Before you can start using your application, your IoC container needs some bindings to be registered.  This is where `Bootstrapper::registerBindings()` comes in handy.  Anything that needs to be bound to the IoC container should be done here.  Once the application is started, all bootstrappers' bindings are registered.

```php
use MyApp\UserRepo;
use Opulence\Ioc\Bootstrappers\Bootstrapper;
use Opulence\Ioc\IContainer;

class MyBootstrapper extends Bootstrapper
{
    public function registerBindings(IContainer $container)
    {
        $container->bindInstance(UserRepo::class, new UserRepo());
    }
}
```

<h2 id="running-bootstrappers">Running Bootstrappers</h2>
Bootstrappers also support a `run()` command, which is run AFTER all bindings have been registered.  This is useful for any last configuration that needs to be performed before the application is run.  Use type-hinted parameters in `run()` for any dependencies your bootstrapper depends on to run successfully.

```php
use MyApp\Database;
use Opulence\Ioc\Bootstrappers\Bootstrapper;

class MyBootstrapper extends Bootstrapper
{
    public function run(Database $database)
    {
        $database->connect();
    }
}
```

<h2 id="bootstrapper-shutdown">Shutting Down Bootstrappers</h2>
Bootstrappers also support a `shutdown()` command, which is run as a pre-shutdown task on the application.  This is useful for any cleaning up a bootstrapper needs to do, such as writing session data.  Use type-hinted parameters in `shutdown()` for any dependencies your bootstrapper depends on to shutdown successfully.

```php
use MyApp\Database;
use Opulence\Ioc\Bootstrappers\Bootstrapper;

class MyBootstrapper extends Bootstrapper
{
    public function shutdown(Database $database)
    {
        $database->disconnect();
    }
}
```

<h2 id="lazy-bootstrappers">Lazy Bootstrappers</h2>
It's not very efficient to create, register bindings, run, and shut down every bootstrapper in your application when they're not all needed.  Sometimes, you may only like a bootstrapper to be registered/run/shut down if its bindings are required.  This is the purpose of **lazy bootstrappers**.  In Opulence, you can designate a bootstrapper to be lazy-loaded by making it implement `Opulence\Ioc\Bootstrappers\ILazyBootstrapper`, which requires a `getBindings()` method to be defined.  This method should return a list of all classes/interfaces bound to the IoC container by that bootstrapper.  Let's take a look at an example:

```php
namespace MyApp\Application\Bootstrappers;

use MyApp\IPostRepo;
use MyApp\PostRepo;
use Opulence\Ioc\Bootstrappers\Bootstrapper;
use Opulence\Ioc\Bootstrappers\ILazyBootstrapper;
use Opulence\Ioc\IContainer;

class MyBootstrapper extends Bootstrapper implements ILazyBootstrapper
{
    public function getBindings() : array
    {
        return [IPostRepo::class];
    }
    
    public function registerBindings(IContainer $container)
    {
        $container->bindInstance(IPostRepo::class, new PostRepo());
    }
}
```

<h4 id="targeted-bindings">Targeted Bindings</h4>
If you take advantage of [targeted bindings](dependency-injection#targeted-bindings) in your lazy bootstrapper, you must indicate so in `getBindings()` by denoting targeted bindings in the format `[BoundClass => TargetClass]`.  Let's say your repository class looks like this:

```php
namespace MyApp;

use Opulence\Orm\DataMappers\IDataMapper;

class PostRepo implements IPostRepo
{
    private $dataMapper;
    
    public function __construct(IDataMapper $dataMapper)
    {
        $this->dataMapper = $dataMapper;
    }
}
```

Let's suppose you always want your dependency injection container to inject an instance of `MyApp\MyDataMapper` into `PostRepo`.  Here's a bootstrapper that accomplishes this:

```php
namespace MyApp\Application\Bootstrappers;

use MyApp\IPostRepo;
use MyApp\MyDataMapper;
use MyApp\PostRepo;
use Opulence\Ioc\Bootstrappers\Bootstrapper;
use Opulence\Ioc\Bootstrappers\ILazyBootstrapper;
use Opulence\Ioc\IContainer;
use Opulence\Orm\DataMappers\IDataMapper;

class MyBootstrapper extends Bootstrapper implements ILazyBootstrapper
{
    public function getBindings() : array
    {
        return [
            // This is a universal binding
            IPostRepo::class,            
            // This is a targeted binding
            [IDataMapper::class => PostRepo::class]
        ];
    }
    
    public function registerBindings(IContainer $container)
    {
        $container->bindInstance(IPostRepo::class, new PostRepo());
        $container->for(PostRepo::class, function (IContainer $container) {
            $container->bindSingleton(IDataMapper::class, MyDataMapper::class);
        });
    }
}
```

`[IDataMapper::class => PostRepo::class]` in `getBindings()` lets the bootstrapper know that `IDataMapper` is bound for `PostRepo`.  When the bootstrapper's bindings are registered, `IDataMapper` will be bound to `MyDataMapper` whenever `PostRepo` is instantiated by the dependency injection container.

<h4 id="bootstrapper-caching">Caching</h4>
Opulence automatically caches data about its lazy and eager (ie not lazy) bootstrappers.  This way, it doesn't have to instantiate each bootstrapper to determine which kind it is.  It also remembers which classes are bound by which bootstrappers.  If you add/remove/modify any bootstrappers, you must run [`php apex framework:flushcache`](console-basics#frameworkflushcache) command in the console to flush this cache.

<h2 id="environment-variables">Environment Variables</h2>
Sometimes, your bootstrappers need access to environment variables.  To access them, simply use PHP's built-in `getenv()` function.

<h2 id="config-values">Config Values</h2>
If you're using the <a href="https://github.com/opulencephp/Project" target="_blank">skeleton project</a> and you need to access config values such as the path to a particular directory, use `Opulence\Framework\Configuration\Config::get($category, $name)`.  `$category` refers to the category of the config value, eg "paths" or "routing".  The `$name` refers to the actual name of the config value.

> **Note:** `Config` is only meant to be used within bootstrappers as a means to grab config values for your application.  It's good practice to never use them outside of bootstrappers.