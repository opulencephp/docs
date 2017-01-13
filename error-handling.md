# Error Handling

## Table of Contents
1. [Introduction](#introduction)
2. [Exception Handlers](#exception-handlers)
  1. [Logging](#logging)
      1. [Specifying Exceptions to Not Log](#specifying-exceptions-to-not-log)
3. [Exception Renderers](#exception-renderers)
  1. [HTTP Responses](#http-responses)
      1. [Response Formats](#response-formats)
4. [Error Handlers](#error-handlers)
  1. [Specifying Errors to Log](#specifying-errors-to-log)
  2. [Specifying Errors to Throw](#specifying-errors-to-throw)

<h2 id="introduction">Introduction</h2>
If you've ever written a PHP page and had some sort of error or unhandled exception, you've probably seen a blank white page in your browser.  Obviously, this is not useful for end users, nor is it helpful for developers when trying to track down the problem.  Opulence's `Debug` library makes it possible to handle errors and exceptions and create useful HTTP responses from them.

<h2 id="exception-handlers">Exception Handlers</h2>
The exception handler is your last line of defense for unhandled exceptions.  Opulence provides `Opulence\Debug\Exceptions\Handlers\ExceptionHandler` as an exception handler. It has two methods:

* `handle()`
  * Handles the `Exception` (or `Throwable` in PHP 7)
  * Useful for logging the exception and rendering a response
* `register()`
  * Registers the handler with PHP

<h4 id="logging">Logging</h4>
`ExceptionHandler` accepts a PSR-3 logger to actually log any errors.  We recommend the excellent <a href="https://github.com/Seldaek/monolog" target="_blank" title="Monolog">Monolog</a> logger.

```php
use Monolog\Handler\ErrorLogHandler;
use Monolog\Logger;
use MyApp\Debug\MyExceptionRenderer;
use Opulence\Debug\Exceptions\Handlers\ExceptionHandler;

$logger = new Logger('app');
$logger->pushHandler(new ErrorLogHandler());
$renderer = new MyExceptionRenderer();
$exceptionHandler = new ExceptionHandler($logger, $renderer);
// Make sure to actually register this handler with PHP
$exceptionHandler->register();
```

<h5 id="specifying-exceptions-to-not-log">Specifying Exceptions to Not Log</h5>
`ExceptionHandler` accepts an array of classes to not log when handled:

```php
$exceptionHandler = new ExceptionHandler($logger, $renderer, [HttpException::class]);
```

Now, whenever an unhandled `HttpException` occurs, it will be handled by the `ExceptionHandler`, but not logged.  All other exceptions will be logged by `$logger`.

<h2 id="exception-renderers">Exception Renderers</h2>
When an exception is handled, you probably want to render some sort of output explaining what happened.  Depending on the environment you're in, you may even want to include technical details to help track down the issue.  This is where exception renderers come in handy.  They must implement `Opulence\Debug\Exceptions\Handlers\IExceptionRenderer`, which has a single method `render()`.  This accepts the `Exception` or `Throwable` and renders some sort of response with it.

<h4 id="http-responses">HTTP Responses</h4>
To compile an HTTP response from an exception, you have two choices:

1. `Opulence\Debug\Exceptions\Handlers\Http\ExceptionRenderer`
2. `Opulence\Framework\Debug\Exceptions\Handlers\Http\ExceptionRenderer`

The first one is useful if you're using the `Debug` library without using the whole Opulence framework.  It can display one page with information about the exception when in the development environment.  It'll display another when in the production environment.  To override the page templates, simply extend `ExceptionRenderer` and override `getDevelopmentEnvironmentContent()` and/or `getProductionEnvironmentContent()`.

The second renderer is useful if you're using the entire Opulence framework.  It'll look for [Fortune](view-fortune) templates for the errors before resorting to the page templates described above.  For example, if an `HttpException` is thrown with a 404 status code, this renderer will look for a template in your *resources/views/errors/html* directory named *404.fortune.php*.

> **Note:** Non-HTTP exceptions will look for a *500.fortune.php* template file.

Two variables will be injected into your Fortune template:

1. `$__exception`
  * The exception object being rendered
2. `$__inDevelopmentEnvironment`
  * Whether or not we are in the development environment

<h5 id="response-formats">Response Formats</h5>
If a user is requesting JSON, it'd be nice to return a formatted JSON response when errors occur.  Opulence provides [Fortune](view-fortune) templates for JSON errors in the skeleton project under the `resources/views/errors/json` directory.

Even if you're using the standalone `Debug` library, Opulence will format your response to match the requested content type.

> **Note:** If the user is not requesting JSON, then the appropriate template under `resources/views/errors/html` will be used.

<h2 id="error-handlers">Error Handlers</h2>
The error handler handles any errors PHP might throw, such as `E_PARSE` or `E_ERROR`.  It even handles fatal errors.  Error handlers have two methods:

1. `handle()`
  * Converts the error into an exception, which is handled by the exception handler
2. `handleShutdown()`
  * Handles any errors after script execution finishes
3. `register()`
  * Registers the handler with PHP

`Opulence\Debug\Errors\Handlers\ErrorHandler` is the default error handler.

<h4 id="specifying-errors-to-log">Specifying Errors to Log</h4>
By default, errors are not logged, although they might be if they're thrown as exceptions.  To actually log certain levels of errors, pass in the bitwise value indicating the levels to log:

```php
use Monolog\Handler\ErrorLogHandler;
use Monolog\Logger;
use Opulence\Debug\Errors\Handlers\ErrorHandler;

$logger = new Logger('app');
$logger->pushHandler(new ErrorLogHandler());
// Assume the exception handler has already been set
$errorHandler = new ErrorHandler(
    $logger,
    $exceptionHandler,
    E_PARSE | E_ERROR
);
```

Now, `E_PARSE` and `E_ERROR` error levels will be logged.

<h4 id="specifying-errors-to-throw">Specifying Errors to Throw</h4>
Errors can be re-thrown as an `\ErrorException`.  This allows them to be handled by the `ExceptionHandler`.  To specify which levels of errors to re-throw as exceptions, pass in the bitwise value indicating the levels to throw:

```php
$errorHandler = new ErrorHandler(
    $logger,
    $exceptionHandler,
    null,
    E_ALL & ~E_NOTICE
);
```

Now, all errors except `E_NOTICE` will be thrown as exceptions.

> **Note:** All errors besides `E_DEPRECATED` and `E_USER_DEPRECATED` are thrown as an `\ErrorException`.
