# Bootstrappers

## Table of Contents
1. [Introduction](#introduction)
2. [Registering Bindings](#registering-bindings)
3. [Inspecting Bindings](#inspecting-bindings)
4. [Environment Variables](#environment-variables)
5. [Config Values](#config-values)

<h2 id="introduction">Introduction</h2>

Most applications need to do some configuration before starting.  For example, they might need to setup a database connection, configure which view engine to use, or assign the authentication scheme to use.  Because Opulence uses [dependency injection](ioc-container) heavily, it's important that you set your bindings in the IoC container.  Bootstrappers are the place to do this.
  
Bootstrappers are loaded before the request is handled.  Typically, they register bindings to the IoC container, but they can also perform any necessary logic once all of the bindings are registered or before the application is shut down.  Bootstrappers extend `Opulence\Ioc\Bootstrappers\Bootstrapper`.  Their constructors are empty and final.

<h2 id="registering-bindings">Registering Bindings</h2>

Before you can start using your application, your IoC container needs some bindings to be registered.  This is where `Bootstrapper::registerBindings()` comes in handy.  Anything that needs to be bound to the IoC container should be done here.  Once the application is started, all bootstrappers' bindings are registered.

```php
use App\Domain\Users\UserRepo;
use Opulence\Ioc\Bootstrappers\Bootstrapper;
use Opulence\Ioc\IContainer;

class PostBootstrapper extends Bootstrapper
{
    public function registerBindings(IContainer $container): void
    {
        $container->bindInstance(UserRepo::class, new UserRepo());
    }
}
```

<h2 id="inspecting-bindings">Inspecting Bindings</h2>

Rather than having to dispatch _every_ bootstrapper on every request, you can use binding inspections to make them dispatched lazily, eg only when they're actually needed.  At a high level, Opulence can look inside your bootstrappers and determine what each of them bind.  It can then set up a factory for each of those bindings that runs `Bootstrapper::registerBindings()` only when at least one of the bootstrapper's bindings is necessary.  This prevents you from having to list out the bindings a bootstrapper registers to get this sort of functionality (like other frameworks force you to do).

Let's we have the following bootstrapper:

```php
namespace Project\Application\Bootstrappers;

use App\Domain\Posts\IPostRepo;
use App\Domain\Posts\PostRepo;
use Opulence\Ioc\Bootstrappers\Bootstrapper;
use Opulence\Ioc\IContainer;

class PostBootstrapper extends Bootstrapper
{
    public function registerBindings(IContainer $container): void
    {
        $container->bindInstance(IPostRepo::class, new PostRepo());
    }
}
```

Let's set up our application to inspect the bindings:

```php
use Opulence\Ioc\Bootstrappers\Inspection\BindingInspectorBootstrapperDispatcher;
use Opulence\Ioc\Bootstrappers\Inspection\Caching\FileBootstrapperBindingCache;
use Opulence\Ioc\Container;

$bootstrappers = [
    new PostBootstrapper()
];
$container = new Container();
$bootstrapperDispatcher = new BindingInspectorBootstrapperDispatcher(
    $container,
    getenv('ENV_NAME') === 'production'
        ? new FileBootstrapperBindingCache('/tmp/bootstrapperInspections.txt')
        : null
);
$bootstrapperDispatcher->dispatch($bootstrappers);
```

That's it.  Now, whenever we call `$container->resolve(IPostRepo::class)`, it will automatically run `PostBootstrapper::registerBindings()` once and use the binding defined inside to resolve it every time after.

> **Note:** It's recommended that you only use caching for bootstrapper bindings in production environments.  Otherwise, changes you make to your bootstrappers might not be reflected.

<h2 id="environment-variables">Environment Variables</h2>

Sometimes, your bootstrappers need access to environment variables.  To access them, simply use PHP's built-in `getenv()` function.

<h2 id="config-values">Config Values</h2>

If you're using the <a href="https://github.com/opulencephp/Project" target="_blank">skeleton project</a> and you need to access config values such as the path to a particular directory, use `Opulence\Framework\Configuration\Config::get($category, $name)`.  `$category` refers to the category of the config value, eg "paths" or "routing".  The `$name` refers to the actual name of the config value.

> **Note:** `Config` is only meant to be used within bootstrappers as a means to grab config values for your application.  It's good practice to never use them outside of bootstrappers.
