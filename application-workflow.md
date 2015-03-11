# Application Workflow

## Table of Contents
1. [HTTP Workflow](#http-workflow)
2. [Console Workflow](#console-workflow)

<h2 id="http-workflow">HTTP Workflow</h2>
RDev uses a single point of entry for all pages.  In other words, all HTTP requests get redirected through `index.php`, which instantiates the application and handles the request.  Here's a breakdown of the workflow of a typical RDev HTTP application:

1. User requests http://www.example.com/users/23/profile
2. `.htaccess` redirects the request through http://www.example.com/index.php
3. `bootstrap/http/start.php` is loaded, which instantiates an `Application` object
4. Various configs are read, and [bootstrappers](bootstrappers) are registered to the `Application`
5. [Pre-start tasks](application#pre-start-tasks) are run
  1. Bootstrappers' bindings are registered
  2. Bootstrappers are run
6. The application is [started](application#start-task)
7. An HTTP `Kernel` is instantiated, which converts the [HTTP request](http#requests) into a [response](http#responses)
  * The path "/users/23/profile" is detected by the request
8. All global [middleware](http-middleware) are run
9. The [`Router`](routing) finds a route that matches the request
  * The user Id 23 is extracted from the URL here
10. The `Dispatcher` runs any [middleware](http-middleware) registered specifically to this route
11. The `Dispatcher` dispatches the request to the `Controller`
  * The user Id is injected into the controller method
12. The `Controller` processes data from the request, updates/retrieves any appropriate models, and creates a `Response`
13. The `Response` is sent back to the user
14. [Post-start tasks](application#post-start-tasks) are run
15. [Pre-shutdown tasks](application#pre-shutdown-tasks) are run
16. The application is [shut down](application#shutdown-task)
17. [Post-shutdown tasks](application#post-shutdown-tasks) are run

<h2 id="console-workflow">Console Workflow</h2>
RDev uses a single point of entry for console applications.  When typing `php rdev` into the console, the `rdev` bash file is executed, which instantiates the application and handles the request.  Here's a breakdown of the workflow of a typical RDev console application:

1. User types `php rdev foo bar --baz=blah`
2. `rdev` bash script loads `bootstrap/console/start.php`, which instantiates an `Application` object
3. Various configs are read, and [bootstrappers](bootstrappers) are registered to the `Application`
4. [Pre-start tasks](application#pre-start-tasks) are run
  1. Bootstrappers' bindings are registered
  2. Bootstrappers are run
5. The application is [started](application#start-task)
6. A console `Kernel` is instantiated with a request parser, the raw string input, and the [response](console#responses) to write output to
7. The `Kernel` uses the request parser to parse the input into a `Request` object
  * This contains the command name ("foo") and any arguments ("bar") and options ("--baz=blah")
8. The command class whose `name` property matches the input command name is instantiated
9. A command compiler compiles the command with the `Request`
10. The command is executed
11. A response compiler compiles any output produced by the command
  * A lexer lexes the raw response into a stream of tokens
  * A parser converts the tokens into an abstract syntax tree
  * The compiler compiles the abstract syntax tree into ANSI codes used to style the output
12. Output is written to the kernel's `Response`, and the command's status code is returned
13. [Post-start tasks](application#post-start-tasks) are run
14. [Pre-shutdown tasks](application#pre-shutdown-tasks) are run
15. The application is [shut down](application#shutdown-task)
16. [Post-shutdown tasks](application#post-shutdown-tasks) are run