# Application

## Table of Contents
1. [Introduction](#introduction)
2. [Kernels](#kernels)
3. [Logging](#logging)
4. [Starting and Shutting Down An Application](#starting-and-shutting-down-an-application)

<h2 id="introduction">Introduction</h2>
An **RDev** application is started up through the `Application` class.  In it, you can configure things like the environment you're on (eg "development" or "production") as well as pre-/post-start and -shutdown tasks to run.

<h2 id="kernels">Kernels</h2>
A kernel is something that takes input, performs processing on it, and returns output.  In RDev, there are two kernels:

1. `RDev\HTTP\Kernels\Kernel`
  * [Read about HTTP applications' workflows](http-workflow)
2. `RDev\Console\Kernels\Kernel`
  * [Read about console applications' workflows](console-workflow)

Having these two kernels allows RDev to function as both a traditional HTTP web application and a console application.

<h2 id="logging">Logging</h2>
RDev takes advantage of the wonderful `Monolog` library.  To learn more about it, <a href="https://github.com/Seldaek/monolog" target="_blank">read its official documentation</a>.

<h2 id="starting-and-shutting-down-an-application">Starting And Shutting Down An Application</h2>
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