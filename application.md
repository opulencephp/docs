# Application

## Table of Contents
1. [Introduction](#introduction)
2. [Workflow](#workflow)
3. [Kernels](#kernels)
4. [Environment](#environment)
  1. [Config Structure](#config-structure)
  2. [Environment Variables](#environment-variables)
5. [Bootstrappers](#bootstrappers)
  1. [Registering Bindings](#registering-bindings)
  2. [Running Bootstrappers](#running-bootstrappers)
  3. [Bootstrapper Example](#bootstrapper-example)
6. [Starting and Shutting Down An Application](#starting-and-shutting-down-an-application)

## Introduction
An **RDev** application is started up through the `Application` class.  In it, you can configure things like the environment you're on (eg "development" or "production") as well as pre-/post-start and -shutdown tasks to run.

## Workflow
RDev uses a single point of entry for all pages.  In other words, all HTTP requests get redirected through `index.php`, which instantiates the application and handles the request.  Here's a breakdown of the workflow of a typical RDev application:

1. User requests http://www.example.com/users/23/profile
2. `.htaccess` file redirects the request through http://www.example.com/index.php
3. `bootstrap/http/start.php` is loaded, which instantiates an `Application` object
4. An HTTP `Kernel` is instantiated, which converts the HTTP request into a response
  * The path "/users/23/profile" is detected by the request
5. The `Router` finds a route that matches the request
  * The user Id 23 is extracted from the URL here
6. The `Dispatcher` dispatches the request to the `Controller`
7. The `Controller` processes data from the request, updates/retrieves any appropriate models, and creates a `Response`
8. The `Response` is sent back to the user and the application is shut down

## Kernels
A kernel is something that takes input, performs processing on it, and returns output.  In RDev, there are 2 kernels:

1. `RDev\Console\Kernels\Kernel`
2. `RDev\HTTP\Kernels\Kernel`

Having these two kernels allows RDev to function as both a console application and a traditional HTTP web application.

## Environment
Sometimes, you might want to change the way your application behaves depending on whether or not it's running on a production, staging, testing, or development machine.  A common example is a database connection - each environment might have different server credentials.  By detecting the environment, you can load the appropriate credentials.  To actually detect the environment, use an `EnvironmentDetector`.  In it, you can specify rules for various environment names.  You can also detect if you're running in a console vs an HTTP connection.

#### Config Structure
The configuration that's passed into `EnvironmentDetector::detect()` should be either:

* A callback function that returns the name of the environment the current server resides in OR
* An array that maps environment names to rules
  * Each rule must be one of the following:
    1. A server host IP or array of host IPs that belong to that environment
    2. An array containing the following keys:
      * "type" => One of the following values:
        * "regex" => Denotes that the rule uses a regular expression
      * "value" => The value of the rule, eg the regular expression to use

Let's take a look at an example:
```php
use RDev\Applications\Environments;

$configArray = [
   // Let's say that there's only one production server
   "production" => "123.456.7.8",
   // Let's say there's a list of staging servers
   "staging" => ["123.456.7.9", "123.456.7.10"],
   // Let's use a regular expression to detect a development environment
   "development" => [
       ["type" => "regex", "value" => "/^192\.168\..*$/"]
   ]
];
$detector = new Environments\EnvironmentDetector($configArray);
$environmentName = $detector->getName();
$environment = new Environments\Environment($environmentName);
```
The following is an example with a custom callback:
```php
use RDev\Applications\Environments;

$callback = function()
{
    // Look to see if a PHP environment variable was set
    if(isset($_ENV["environment"]))
    {
        return $_ENV["environment"];
    }

    // By default, return production
    return "production";
};
$detector = new Environments\EnvironmentDetector($callback);
$environmentName = $detector->getName();
$environment = new Environments\Environment($environmentName);
```

#### Environment Variables
Variables that are specifically tied to the environment the application is running on are called *environment variables*.  Setting an environment variable using RDev is as easy as `$environment->setVariable("foo", "bar")`.  To make configuring your environment variables as easy as possible, RDev supports environment config files, whose names are of the format ".env.DESCRIPTION_OF_CONFIG.php".  They should exist in your "configs/environment" directory.  These files are automatically run before the application is booted up.  Let's take a look at an example:
 
##### .env.example.php
```php
$environment->setVariable("DB_HOST", "localhost");
$environment->setVariable("DB_USER", "myuser");
$environment->setVariable("DB_PASSWORD", "mypassword");
$environment->setVariable("DB_NAME", "public");
$environment->setVariable("DB_PORT", 5432);
```

> **Note:** For performance reasons, .env.*.php files are only loaded on non-production servers.  It is strongly recommended that production servers are setup with hard-coded environment variables in their configs.  For security, it's strongly recommended that you do not version-control your environment variable configs.  Instead, each developer should be given a template of the environment config, and should fill out the config with the appropriate values for their environment.

## Bootstrappers
Most applications need to do some configuration before starting.  A common task is registering bindings, and yet another is setting up database connections.  You can do this bootstrapping by extending `RDev\Applications\Bootstrappers\Bootstrapper`.  It accepts `Paths`, `Environment`, and `ISession` objects in its constructor, which can be useful for something like binding a particular database instance based on the current environment.

#### Registering Bindings
Before you can start using your application, your IoC container needs some bindings to be registered.  This is where `Bootstrapper::registerBindings()` comes in handy.  Anything that needs to be bound to the IoC container should be done here.  Once the application is started, all bootstrappers' bindings are registered.

#### Running Bootstrappers
Bootstrappers also support a `run()` command, which is run AFTER all bindings have been registered.  This is useful for any last configuration that needs to be performed before the application is run.  Use type-hinted parameters in `run()` for any dependencies your bootstrapper depends on to run successfully.

#### Bootstrapper Example
Let's pretend you're developing an application grabs WordPress posts from a database and displays them in a nice view.  You might have a `Posts` class, which needs a database connection to read the posts.  Let's take a look at a simple bootstrapper:

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

## Starting And Shutting Down An Application
To start and shutdown an application, simply call the `start()` and `shutdown()` methods, respectively, on the application object.  If you'd like to do some tasks before or after startup, you may do them using `registerPreStartTask()` and `registerPostStartTask()`, respectively.  Similarly, you can add tasks before and after shutdown using `registerPreShutdownTask()` and `registerPostShutdownTask()`, respectively.  These tasks are handy places to do any setting up that your application requires or any housekeeping after start/shutdown.

Let's look at an example of using these tasks:
```php
// Let's register a pre-start task
$application->registerPreStartTask(function()
{
    error_log("Application issued start command at " . date("Y-m-d H:i:s"));
});
// Let's register a post-start task
$application->registerPostStartTask(function()
{
    error_log("Application started at " . date("Y-m-d H:i:s"));
});
// Let's register a pre-shutdown task
$application->registerPreShutdownTask(function()
{
    error_log("Application issued shutdown command at " . date("Y-m-d H:i:s"));
});
// Let's register a post-shutdown task
$application->registerPostShutdownTask(function()
{
    error_log("Application shutdown at " . date("Y-m-d H:i:s"));
});
$application->start();
$application->shutdown();
```