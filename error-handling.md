# Error Handling

## Table of Contents
1. [Introduction](#introduction)
2. [Exception Handlers](#exception-handlers)
  1. [Logging](#logging)
3. [Exception Renderers](#exception-renderers)
  1. [HTTP Responses](#http-responses)
4. [Error Handlers](#error-handlers)

<h2 id="introduction">Introduction</h2>
If you've ever written a PHP page and had some sort of error or unhandled exception, you've probably seen a blank white page in your browser.  Obviously, this is not useful for end users, nor is it helpful for developers when trying to track down the problem.  Opulence's `Debug` library makes it possible to handle errors and exceptions and create useful HTTP responses from them.
 
<h2 id="exception-handlers">Exception Handlers</h2>
The exception handler is your last line of defense for unhandled exceptions.  Opulence provides `Opulence\Debug\Exceptions\Handlers\ExceptionHandler` as an exception handler. It has two methods:
 
* `handle()`
  * Handles the `Exception` (or `Throwable` in PHP 7)
  * Useful for logging the error and rendering a response
* `register()`
  * Registers the handler with PHP
  
<h3 id="logging">Logging</h3>
`ExceptionHandler` accepts a PSR-3 logger to actually log any errors.  We recommend the excellent <a href="https://github.com/Seldaek/monolog" target="_blank" title="Monolog">Monolog</a> logger.

You might not want to log all exceptions (such as `HttpException`).  In this case, you can specify which exceptions to not log in the last parameter of the constructor:

```php
use Monolog\Handler\ErrorLogHandler;
use Monolog\Logger;
use MyApp\Debug\MyExceptionRenderer;
use Opulence\Debug\Exceptions\Handlers\ExceptionHandler;
use Opulence\Http\HttpException;

$logger = new Logger("app");
$logger->pushHandler(new ErrorLogHandler());
$renderer = new MyExceptionRenderer();
$exceptionHandler = new ExceptionHandler($logger, $renderer, [HttpException::class]);
// Make sure to actually register this handler with PHP
$exceptionHandler->register();
```

Now, whenever an unhandled `HttpException` occurs, it will be handled by the `ExceptionHandler`, but not logged.  All other exceptions will be logged by `$logger`.

<h2 id="exception-renderers">Exception Renderers</h2>
When an exception is handled, you probably want to render some sort of output explaining what happened.  Depending on the environment you're in, you may even want to include technical details to help track down the issue.  This is where exception renderers come in handy.  They must implement `Opulence\Debug\Exceptions\Handlers\IErrorRenderer`, which has a single method `render($ex)`.

<h3 id="http-responses">HTTP Responses</h3>
To compile an HTTP response from an exception, you have two choices:

1. `Opulence\Debug\Exceptions\Handlers\Http\ExceptionRenderer`
2. `Opulence\Framework\Debug\Exceptions\Handlers\Http\ExceptionRenderer`

The first one is useful if you're using the `Debug` library without using the whole Opulence framework.  It can display one page with information about the exception when in the development environment.  It'll display another when in the production environment.  To override the page templates, simply extend `ExceptionRenderer` and override `getDevelopmentEnvironmentContent()` and/or `getProductionEnvironmentContent()`.

The second renderer is useful if you're using the entire Opulence framework.  It'll look for [Fortune](view-fortune) templates for the errors before resorting to the page templates described above.  For example, if an `HttpException` is thrown with a 404 status code, this renderer will look for a template in your `views` directory named `404.fortune.php`.

> **Note:** Non-HTTP exceptions will look for a `500.fortune.php` template file.

Two variables will be injected into your Fortune template:

1. `$__exception`
  * The exception object being rendered
2. `$__inDevelopmentEnvironment`
  * Whether or not we are in the development environment

<h2 id="error-handlers">Error Handlers</h3>
The error handler handles any errors PHP might throw, such as `E_PARSE` or `E_ERROR`.  It even handles fatal errors.  Error handlers have two methods:

1. `handle()`
  * Converts the error into an exception, which is handled by the exception handler
2. `register()`
  * Registers the handler with PHP

`Opulence\Debug\Errors\Handlers\ErrorHandler` is the default error handler.