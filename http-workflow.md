# HTTP Workflow

RDev uses a single point of entry for all pages.  In other words, all HTTP requests get redirected through `index.php`, which instantiates the application and handles the request.  Here's a breakdown of the workflow of a typical RDev HTTP application:

1. User requests http://www.example.com/users/23/profile
2. `.htaccess` redirects the request through http://www.example.com/index.php
3. `bootstrap/http/start.php` is loaded, which instantiates an `Application` object
4. Various configs are read, and [bootstrappers](application#bootstrappers) are registered to the `Application`
5. [Pre-start tasks](application#starting-and-shutting-down-an-application) are run
  1. Bootstrappers' bindings are registered
  2. Bootstrappers are run
6. The application is started
7. Post-start tasks are run
8. An HTTP `Kernel` is instantiated, which converts the [HTTP request](http#requests) into a [response](http#responses)
  * The path "/users/23/profile" is detected by the request
9. The [`Router`](routing) finds a route that matches the request
  * The user Id 23 is extracted from the URL here
10. The `Dispatcher` dispatches the request to the `Controller`
11. The `Controller` processes data from the request, updates/retrieves any appropriate models, and creates a `Response`
12. The `Response` is sent back to the user
13. Pre-shutdown tasks are run
14. The application is shut down
15. Post-shutdown tasks are run