# Routing

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Usage](#basic-usage)
  1. [Multiple Methods](#multiple-methods)
3. [Route Variables](#route-variables)
  1. [Regular Expressions](#regular-expressions)
  2. [Optional Variables](#optional-variables)
  3. [Default Values](#default-values)
4. [Host Matching](#host-matching)
5. [Middleware](#middleware)
  1. [Changing the Request](#changing-the-request)
  2. [Changing the Response](#changing-the-response)
6. [HTTPS](#https)
7. [Named Routes](#named-routes)
8. [Route Grouping](#route-grouping)
  1. [Controller Namespaces](#controller-namespaces)
  2. [Group Middleware](#group-middleware)
  3. [Group Hosts](#group-hosts)
  4. [Group HTTPS](#group-https)
  5. [Group Variable Regular Expressions](#group-variable-regular-expressions)
9. [URL Generators](#url-generators)
  1. [Generating URLs from Code](#generating-urls-from-code)
  2. [Generating URLs from Views](#generating-urls-from-views)
10. [Missing Routes](#missing-routes)
11. [Notes](#notes)

<h2 id="introduction">Introduction</h2>
So, you've made some page templates, and you've written some models.  Now, you need a way to wire everything up so that users can access your pages.  To do this, you need a `Router` and controllers.  The `Router` can capture data from the URL to help you decide which controller to use and what data to send to the view.  It makes building a RESTful application a cinch.

<h2 id="basic-usage">Basic Usage</h2>
Routes require a few pieces of information:
* The path the route is valid for
* The HTTP method (eg "GET", "POST", "DELETE", or "PUT") the route is valid for
* The fully-qualified name of the controller class to use
* The name of the method in that controller to call to render the view

Let's take a look at a simple route that maps a GET request to the path "/users":
```php
use RDev\IoC;
use RDev\HTTP\Routing;
use RDev\HTTP\Routing\Compilers;
use RDev\HTTP\Routing\Compilers\Parsers;
use RDev\HTTP\Routing\Dispatchers;

$container = new IoC\Container();
$dispatcher = new Dispatchers\Dispatcher($container);
$compiler = new Compilers\Compiler(new Parsers\Parser());
$router = new Routing\Router($container, $dispatcher, $compiler);
// This will route a GET request to "/users" to MyController->getAllUsers()
$router->get("/users", ["controller" => "MyApp\\MyController@getAllUsers"]);
// This will route a POST request to "/login" to MyController->login()
$router->post("/login", ["controller" => "MyApp\\MyController@login"]);
// This will route a DELETE request to "/users/me" to MyController->deleteUser()
$router->delete("/users/me", ["controller" => "MyApp\\MyController@deleteUser"]);
// This will route a PUT request to "/users/profile/image" to MyController->uploadProfileImage()
$router->put("/users/profile/image", ["controller" => "MyApp\\MyController@uploadProfileImage"]);
```

The router takes advantage of the [Dependency Injection Container](ioc) to instantiate your controller.

> **Note:** Primitives (eg strings and arrays) should not appear in a controller's constructor because the IoC container would have no way of resolving those dependencies at runtime.  Stick to type-hinted objects in the constructors.

<h4 id="multiple-methods">Multiple Methods</h4>
You can register a route to multiple methods using the router's `multiple()` method:
```php
$router->multiple(["GET", "POST"], ["controller" => "MyApp\\MyController@myMethod"]);
```

To register a route for all methods, use the `any()` method:
```php
$router->any(["controller" => "MyApp\\MyController@myMethod"]);
```

<h2 id="route-variables">Route Variables</h2>
Let's say you want to grab a specific user's profile page.  You'll probably want to structure your URL like "/users/{userId}/profile", where "{userId}" is the Id of the user whose profile we want to view.  Using a `Router`, the data matched in "{userId}" will be mapped to a parameter in your controller's method named "$userId".

> **Note**: All non-optional parameters in the controller method must have identically-named route variables.  In other words, if your method looks like `function showBook($authorName, $bookTitle = null)`, your path must have a "{authorName}" variable.  The routes "/authors/{authorName}/books" and "/authors/{authorName}/books/{bookTitle}" would be valid, but "/authors" would not.

Let's take a look at a full example:
```php
use RDev\HTTP\Routing;

class UserController extends Routing\Controller
{
    public function showProfile($userId)
    {
        return "Profile for user " . $userId;
    }
}

$router->get("/users/{userId}/profile", ["controller" => "MyApp\\UserController@showProfile"]);
```

Calling the path "/users/23/profile" will return "Profile for user 23".

<h4 id="regular-expressions">Regular Expressions</h4>
If you'd like to enforce certain rules for a route variable, you may do so in the options array.  Simply add a "variables" entry with variable names-to-regular-expression mappings:
```php
$router->get("/users/{userId}/profile", [
    "controller" => "MyApp\\UserController@showProfile",
    "variables" => [
        "userId" => "\d+" // The user Id variable must now be a number
    ]
]);
```

<h4 id="optional-variables">Optional Variables</h4>
If a certain variable is optional, simply append "?" to it:
```php
$router->get("/books/{bookId?}", ["controller" => "MyApp\\BookController@showBook"]);
```

This would match both "/books/" and "/books/23".

<h4 id="default-values">Default Values</h4>
Sometimes, you might want to have a default value for a route variable.  Doing so is simple:
```php
$router->get("/food/{foodName=all}", ["controller" => "MyApp\\FoodController@showFood"]);
```

If no food name was specified, "all" will be the default value.

> **Note:** To give an optional variable a default value, structure the route variable like "{varName?=value}".

<h2 id="host-matching">Host Matching</h2>
Routers can match on hosts as well as paths.  Want to match calls to a subdomain?  Easy:

```php
$routeOptions = [
    "controller" => "MyApp\\InboxController@showInbox",
    "host" => "mail.mysite.com" 
];
$router->get("/inbox", $routeOptions);
```

Just like with paths, you can create variables from components of your host.  In the following example, a variable called `$subdomain` will be passed into `MyApp\SomeController::doSomething()`:

```php
$routeOptions = [
    "controller" => "MyApp\\SomeController@doSomething",
    "host" => "{subdomain}.mysite.com" 
];
$router->get("/foo", $routeOptions);
```

Host variables can also have regular expression constraints, similar to path variables.

<h2 id="middleware">Middleware</h2>
Some routes might require actions to occur before and after the controller is called.  For example, you might want to check if a user is authenticated before allowing him or her access to a certain page.  This is where `Middleware` comes in handy.  `Middleware` are a series of functions that manipulate the `Request` and `Response`.  They are executed in series in a [Pipeline](pipelines).  Let's take a look at an example:

```php
use MyApp\Authentication;
use RDev\HTTP\Middleware;
use RDev\HTTP\Requests;
use RDev\HTTP\Responses;

class Authentication implements Middleware\IMiddleware
{
    private $authenticator = null;
    
    // Inject any dependencies your middleware needs
    public function __construct(Authentication\Authenticator $authenticator)
    {
        $this->authenticator = $authenticator;
    }

    // $next consists of the next middleware in the pipeline
    public function handle(Requests\Request $request, \Closure $next)
    {
        if(!$this->authenticator->isLoggedIn())
        {
            return new Responses\RedirectResponse("/login");
        }
        
        return $next($request);
    }
}

// Add this middleware to a route
$router->post("/users/posts", [
    "controller" => "MyApp\\UserController@createPost",
    "middleware" => "MyApp\\Authenticate" // Could also be an array of middleware
])
```

Now, the `Authenticate` middleware will be run before the `createPost()` method is called.  If the user is not logged in, he'll be redirected to the login page.

> **Note:** If middleware does not specifically call the `$next` closure, none of the middleware after it in the pipeline will be run.

<h4 id="changing-the-request">Changing the Request</h4>
To manipulate the request before it gets to the controller, make changes to it before calling `$next($request)`:

```php
use RDev\HTTP\Middleware;

class RequestManipulator implements Middleware\IMiddleware
{
    public function handle(Requests\Request $request, \Closure $next)
    {
        // Do our work before returning $next($request)
        $request->getHeaders()->add("SOME_HEADER", "foo");
        
        return $next($request);
    }
}
```

<h4 id="changing-the-response">Changing the Response</h4>
To manipulate the response after the controller has done its work, do the following:

```php
use RDev\HTTP\Middleware;
use RDev\HTTP\Responses;

class ResponseManipulator implements Middleware\IMiddleware
{
    public function handle(Requests\Request $request, \Closure $next)
    {
        $response = $next($request);
        
        // Make our changes
        $cookie = new Responses\Cookie("my_cookie", "foo", \DateTime::createFromFormat("+1 week"));
        $response->getHeaders()->setCookie($cookie);
        
        return $response;
    }
}
```

<h2 id="https">HTTPS</h2>
Some routes should only match on an HTTPS connection.  To do this, set the `https` flag to true in the options:

```php
$options = [
    "controller" => "MyApp\\MyController@myMethod",
    "https" => true
];
$router->get("/users", $options);
```

HTTPS requests to "/users" will match, but non SSL connections will return a 404 response.

<h2 id="named-routes">Named Routes</h2>
Routes can be given a name, which makes them identifiable.  This is especially useful for things like [generating URLs for a route](#genearting-urls-from-code).  To name a route, pass a `"name" => "THE_NAME"` into the route options:

```php
$options = [
    "controller" => "MyApp\\MyController@myMethod",
    "name" => "awesome"
];
$router->get("/users", $options);
```

This will create a route named "awesome".

<h2 id="route-grouping">Route Grouping</h2>
One of the most important sayings in programming is "Don't repeat yourself" or "DRY".  In other words, don't copy-and-paste code because that leads to difficulties in maintaining/changing the code base in the future.  Let's say you have several routes that start with the same path.  Instead of having to write out the full path for each route, you can create a group:
```php
$router->group(["path" => "/users/{userId}"], function() use ($router)
{
    $router->get("/profile", ["controller" => "MyApp\\UserController@showProfile"]);
    $router->delete("", ["controller" => "MyApp\\UserController@deleteUser"]);
});
```

Now, a GET request to "/users/{userId}/profile" will get a user's profile, and a DELETE request to "/users/{userId}" will delete a user.

<h4 id="controller-namespaces">Controller Namespaces</h4>
If all the controllers in a route group belong under a common namespace, you can specify the namespace in the group options:
```php
$router->group(["controllerNamespace" => "MyApp\\Controllers"], function() use ($router)
{
    $router->get("/users", ["controller" => "UserController@showAllUsers"]);
    $router->get("/posts", ["controller" => "PostController@showAllPosts"]);
});
```

Now, a GET request to "/users" will route to `MyApp\Controllers\UserController::showAllUsers()`, and a GET request to "/posts" will route to `MyApp\Controllers\PostController::showAllPosts()`.

<h4 id="group-middleware">Group Middleware</h4>
Route groups allow you to apply middleware to multiple routes:
```php
$router->group(["middleware" => "MyApp\\Authenticate"], function() use ($router)
{
    $router->get("/users/{userId}/profile", ["controller" => "MyApp\\UserController@showProfile"]);
    $router->get("/posts", ["controller" => "MyApp\\PostController@showPosts"]);
});
```

The `Authenticate` middleware will be executed on any matched routes inside the closure.

<h4 id="group-hosts">Group Hosts</h4>
You can filter by host in router groups:

```php
$router->group(["host" => "google.com"], function() use ($router)
{
    $router->get("/", ["controller" => "MyApp\\HomeController@showHomePage"]);
    $router->group(["host" => "mail."], function() use ($router)
    {
        $router->get("/", ["controller" => "MyApp\\MailController@showInbox"]);
    });
});
```

> **Note:** When specifying hosts in nested router groups, the inner groups' hosts are prepended to the outer groups' hosts.  This means the inner-most route in the example above will have a host of "mail.google.com".

<h4 id="group-https">Group HTTPS</h4>
You can force all routes in a group to be HTTPS:

```php
$router->group(["https" => true], function() use ($router)
{
    $router->get("/", ["controller" => "MyApp\\HomeController@showHomePage"]);
    $router->get("/books", ["controller" => "MyApp\\BookController@showBooksPage"]);
});
```

> **Note:** If the an outer group marks the routes HTTPS but an inner one doesn't, the inner group gets ignored.  The outer-most group with an HTTPS definition is the only one that counts.

<h4 id="group-variable-regular-expressions">Group Variable Regular Expressions</h4>
Groups support regular expressions for path variables:

```php
$router->group(["path" => "/users/{userId}", "variables" => ["userId" => ["\d+"]], function() use ($router)
{
    $router->get("/profile", ["controller" => "MyApp\\ProfileController@showProfilePage"]);
    $router->get("/posts", ["controller" => "MyApp\\PostController@showPostsPage"]);
});
```

Going to "/users/foo/profile" or "users/foo/posts" will not match because the Id was not numeric.

> **Note:** If a route has a variable regular expression specified, it takes precedence over group regular expressions.

<h2 id="missing-routes">Missing Routes</h2>
In the case that the router cannot find a route that matches the request, a 404 response will be returned.  If you'd like to customize your 404 page or any other HTTP error status page, override `showHTTPError()` in your controller and display the appropriate response.  Register your controller in the case of a missing route using `Router::setMissedRouteControllerName()`:

Then, just add a route to handle this:
```php
namespace MyApp;
use RDev\HTTP\Requests;
use RDev\HTTP\Responses;
use RDev\HTTP\Routing;

class MyController extends Routing\Controller
{
    public function showHTTPError($statusCode)
    {
        switch($statusCode)
        {
            case Responses\ResponseHeaders::HTTP_NOT_FOUND:
                return new Responses\Response("My custom 404 page", $statusCode);
            default:
                return new Responses\Response("Something went wrong", $statusCode);
        }
    }
}

$router->setMissedRouteControllerName("MyApp\\MyController");
// Assume $request points to a request object with a path that isn't covered in the router
$router->route($request); // Returns a 404 response with "My custom 404 page"
```

<h2 id="url-generators">URL Generators</h2>
A cool feature is the ability to generate URLs from named routes using `RDev\HTTP\Routing\URL\URLGenerator`.  If your route has variables in the domain or path, you just pass them in `URLGenerator::createFromName()`.  Unless a host is specified in the route, an absolute path is generated.  Secure routes with hosts specified will generate `https://` absolute URLs.

> **Note:** If you do not define all the non-optional variables in the host or domain, a `URLException` will be thrown.

<h4 id="generating-urls-from-code">Generating URLs from Code</h4>
```php
use RDev\HTTP\Routing;
use RDev\HTTP\Routing\Compilers;
use RDev\HTTP\Routing\Compilers\Parsers;
use RDev\HTTP\Routing\URL;

// Let's assume the router is already instantiated
$compiler = new Compilers\Compiler(new Parsers\Parser());
$urlGenerator = new URL\URLGenerator($router->getRoutes(), $compiler);
// Let's add a route named "profile"
$router->get("/users/{userId}", ["controller" => "MyApp\\ProfileController@showProfile", "name" => "profile"]);
// Now we can generate a URL and pass in data to it
echo $urlGenerator->createFromName("profile", 23); // "/users/23"
```

If we specify a host in our route, an absolute URL is generated.  We can even define variables in the host:

```php
// Let's assume the URL generator is already instantiated
// Let's add a route named "inbox"
$routeOptions = [
    "controller" => "MyApp\\InboxController@showInbox",
    "host" => "{country}.mail.example.com",
    "name" => "inbox"
];
$router->get("/users/{userId}", $routeOptions);
// Any values passed in will first be used to define variables in the host
// Any leftover values will define the values in the path
echo $urlGenerator->createFromName("inbox", ["us", 724]); // "http://us.mail.example.com/users/724"
```

<h4 id="generating-urls-from-views">Generating URLs from Views</h4>
URLs can also be generated from views using the `route()` template function.  Here's an example router config:

```php
$router->get("/users/{userId}/profile", [
    "controller" => "UserController@showProfile", 
    "name" => "profile"
]);
```

Here's how to generate a URL to the "profile" route:

```php
<a href="{{route('profile', [123])}}">View Profile</a>
```

This will compile to:

```
<a href="/users/123/profile">View Profile</a>
```

<h2 id="notes">Notes</h2>
Routes are matched based on the order they were added to the router.  So, if you did the following:
```php
$router->get("/{foo}", [
    "controller" => "MyApp\\MyController@myMethod",
    "variables" => [
        "foo" => ".*"
    ]
]);
$router->get("/users", ["controller" => "MyApp\\MyController@myMethod"]);
```

...The first route "/{foo}" would always match first because it was added first.  Add any "fall-through" routes after you've added the rest of your routes.