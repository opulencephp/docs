# HTTP Middleware

## Table of Contents
1. [Introduction](#introduction)
2. [Manipulating the Request](#manipulating-the-request)
3. [Manipulating the Response](#manipulating-the-response)
4. [Global Middleware](#global-middleware)
5. [Route Middleware](#route-middleware)
6. [Middleware Parameters](#middleware-parameters)
  
<h2 id="introduction">Introduction</h2>
HTTP middleware are classes that sit in between the `Kernel` and `Controller`.  They manipulate the request and response to do things like authenticate users or enforce CSRF protection for certain routes.  They are executed in series in a [pipeline](pipelines).

Opulence uses dependency injection for type-hinted objects in a `Middleware` constructor.  So, if you need any objects in your `handle()` method, just specify them in the constructor.  Let's take a look at an example:

```php
namespace MyApp;

use Closure;
use MyApp\Authentication\Authenticator;
use Opulence\Http\Middleware\IMiddleware;
use Opulence\Http\Requests\Request;
use Opulence\Http\Responses\RedirectResponse;
use Opulence\Http\Responses\Response;

class Authentication implements IMiddleware
{
    private $authenticator = null;
    
    // Inject any dependencies your middleware needs
    public function __construct(Authenticator $authenticator)
    {
        $this->authenticator = $authenticator;
    }

    // $next consists of the next middleware in the pipeline
    public function handle(Request $request, Closure $next) : Response
    {
        if (!$this->authenticator->isLoggedIn()) {
            return new RedirectResponse("/login");
        }
        
        return $next($request);
    }
}
```

Add this middleware to a route:

```php
$router->post("/users/posts", [
    "controller" => "MyApp\\UserController@createPost",
    "middleware" => "MyApp\\Authentication" // Could also be an array of middleware
]);
```

Now, the `Authenticate` middleware will be run before the `createPost()` method is called.  If the user is not logged in, he'll be redirected to the login page.

> **Note:** If middleware does not specifically call the `$next` closure, none of the middleware after it in the pipeline will be run.

> **Note:** To get the current `Route` (which is accessible through `Router::getMatchedRoute()`), inject `Opulence\Routing\Router` into the constructor.

<h2 id="manipulating-the-request">Manipulating the Request</h2>
To manipulate the request before it gets to the controller, make changes to it before calling `$next($request)`:

```php
use Closure;
use Opulence\Http\Middleware\IMiddleware;
use Opulence\Http\Requests\Request;
use Opulence\Http\Responses\Response;

class RequestManipulator implements IMiddleware
{
    public function handle(Request $request, Closure $next) : Response
    {
        // Do our work before returning $next($request)
        $request->getHeaders()->add("SOME_HEADER", "foo");
        
        return $next($request);
    }
}
```

<h2 id="manipulating-the-response">Manipulating the Response</h2>
To manipulate the response after the controller has done its work, do the following:

```php
use Closure;
use DateTime;
use Opulence\Http\Middleware\IMiddleware;
use Opulence\Http\Requests\Request;
use Opulence\Http\Responses\Cookie;

class ResponseManipulator implements IMiddleware
{
    public function handle(Request $request, Closure $next) : Response
    {
        $response = $next($request);
        
        // Make our changes
        $cookie = new Cookie("my_cookie", "foo", DateTime::createFromFormat("+1 week"));
        $response->getHeaders()->setCookie($cookie);
        
        return $response;
    }
}
```

<h2 id="global-middleware">Global Middleware</h2>
Global middleware is middleware that is run on every route.  To add middleware to the list of global middleware, add the fully-qualified middleware class' name to the array in `config/http/middleware.php`.

<h2 id="route-middleware">Route Middleware</h2>
To learn how to register middleware with routes, read the [routing tutorial](routing#middleware).  You can also learn how to [add middleware to route groups](routing#group-middleware) there.

<h2 id="middleware-parameters">Middleware Parameters</h2>
Occasionally, you'll find yourself wanting to pass in primitive values to middleware to indicate something such as a required role to see a page.  In these cases, your middleware should extend `Opulence\Http\Middleware\ParameterizedMiddleware`:

```php
use Closure;
use Opulence\Http\HttpException;
use Opulence\Http\Middleware\ParameterizedMiddleware;
use Opulence\Http\Requests\Request;
use Opulence\Http\Responses\Response;

class RoleMiddleware extends ParameterizedMiddleware
{
    private $user;

    // Inject any dependencies your middleware needs
    public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function handle(Request $request, Closure $next) : Response
    {
        // Parameters are available in $this->parameters
        if (!$this->user->hasRole($this->parameters["role"])) {
            throw new HttpException(403);
        }
        
        return $next($request);
    }
}
```

To actually specify `role`, use `{Your middleware}::withParameters()` in your router configuration:

```php
$options = [
    "middleware" => RoleMiddleware::withParameters(["role" => "admin"])
];
$router->get("/users", "MyController\\MyController@myMethod", $options);
```