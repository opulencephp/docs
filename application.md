# Application

## Table of Contents
1. [Introduction](#introduction)
2. [Kernels](#kernels)
3. [Logging](#logging)
4. [Tasks](#tasks)
  1. [Pre-Start Tasks](#pre-start-tasks)
  2. [Start Task](#start-task)
  3. [Post-Start Tasks](#post-start-tasks)
  4. [Pre-Shutdown Tasks](#pre-shutdown-tasks)
  5. [Shutdown Task](#shutdown-task)
  6. [Post-Shutdown Tasks](#post-shutdown-tasks)

<h2 id="introduction">Introduction</h2>
An **RDev** application is started up through the `Application` class.  You can register pre-start, post-start, pre-shutdown, and post-shutdown tasks to be run on every request.  You can also register [bootstrappers](bootstrappers) to bind objects to your [dependency injection container](dependency-injection).  This allows you to program to an interface and resolve dependencies at runtime.  Once the application is started, a `Kernel` is instantiated and actually handles the request.

<h2 id="kernels">Kernels</h2>
A kernel is something that takes input, performs processing on it, and returns output.  In RDev, there are two kernels:

1. `RDev\HTTP\Kernels\Kernel`
  * [Read about HTTP applications' workflows](http-workflow)
2. `RDev\Console\Kernels\Kernel`
  * [Read about console applications' workflows](console-workflow)

Having these two kernels allows RDev to function as both a traditional HTTP web application and a console application.

<h2 id="logging">Logging</h2>
RDev takes advantage of the wonderful `Monolog` library.  To learn more about it, <a href="https://github.com/Seldaek/monolog" target="_blank">read its official documentation</a>.

<h2 id="tasks">Tasks</h2>
To start and shutdown an application, simply call the `start()` and `shutdown()` methods, respectively, on the application object.  If you'd like to do some tasks before or after startup, you may do them using `registerPreStartTask()` and `registerPostStartTask()`, respectively.  Similarly, you can add tasks before and after shutdown using `registerPreShutdownTask()` and `registerPostShutdownTask()`, respectively.  These tasks are handy places to do any setting up that your application requires or any housekeeping after start/shutdown.

<h4 id="pre-start-tasks">Pre-Start Tasks</h4>
```php
$application->registerPreStartTask(function()
{
    error_log("Application issued start command at " . date("Y-m-d H:i:s"));
});
```

<h4 id="start-task">Start Task</h4>
If you'd like to perform a certain task after all the pre-start tasks have been completed, pass in a `Closure` to the `start()` method.  This is commonly used to instantiate a `Kernel` and handle a request.
```php
$application->start(function()
{
    error_log("Application actually started at " . date("Y-m-d H:i:s"));
});
```

> **Note:** Passing a `Closure` to `start()` is optional.

<h4 id="post-start-tasks">Post-Start Tasks</h4>
```php
$application->registerPostStartTask(function()
{
    error_log("Application finished starting at " . date("Y-m-d H:i:s"));
});
```

<h4 id="pre-shutdown-tasks">Pre-Shutdown Tasks</h4>
```php
$application->registerPreShutdownTask(function()
{
    error_log("Application issued shutdown command at " . date("Y-m-d H:i:s"));
});
```

<h4 id="shutdown-task">Shutdown Task</h4>
If you'd like to perform a certain task after all the pre-shutdown tasks have been completed, pass in a `Closure` to the `shutdown()` method.
```php
$application->shutdown(function()
{
    error_log("Application actually shutdown at " . date("Y-m-d H:i:s"));
});
```

> **Note:** Passing a `Closure` to `shutdown()` is optional. 

<h4 id="post-shutdown-tasks">Post-Shutdown Tasks</h4>
```php
$application->registerPostShutdownTask(function()
{
    error_log("Application finished shutting down at " . date("Y-m-d H:i:s"));
});
```