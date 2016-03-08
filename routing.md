# Routing

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Usage](#basic-usage)
  1. [Using Closures](#using-closures)
  2. [Using Controller Classes](#using-controller-classes)
  3. [Multiple Methods](#multiple-methods)
3. [Route Variables](#route-variables)
  1. [Regular Expressions](#regular-expressions)
  2. [Optional Parts](#optional-parts)
  3. [Default Values](#default-values)
4. [Host Matching](#host-matching)
5. [Middleware](#middleware)
  1. [Middleware Parameters](#middleware-parameters)
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
10. [Caching](#caching)
11. [Missing Routes](#missing-routes)
10. [Notes](#notes)

<h2 id="introduction">Introduction</h2>
So, you've made some page views, and you've written some models.  Now, you need a way to wire everything up so that users can access your pages.  To do this, you need a `Router` and controllers.  The `Router` can capture data from the URL to help you decide which controller to use and what data to send to the view.  It makes building a RESTful application a cinch.

<h2 id="basic-usage">Basic Usage</h2>
Routes require a few pieces of information:
* The path the route is valid for
* The HTTP method (eg "GET", "POST", "DELETE", "PUT", "PATCH", "OPTIONS", or "HEAD") the route is valid for
* The action to perform on a match

`Opulence\Routing\Router` supports various methods out of the gate:
* `delete()`
* `get()`
* `head()`
* `options()`
* `patch()`
* `post()`
* `put()`

<h4 id="using-closures">Using Closures</h4>
For very simple applications, it's probably easiest to use closures as your routes' controllers:

```php
use Opulence\Ioc\Container;
use Opulence\Routing\Dispatchers\Dispatcher;
use Opulence\Routing\Router;
use Opulence\Routing\Routes\Compilers\Compiler;
use Opulence\Routing\Routes\Compilers\Matchers\HostMatcher;
use Opulence\Routing\Routes\Compilers\Matchers\PathMatcher;
use Opulence\Routing\Routes\Compilers\Matchers\SchemeMatcher;
use Opulence\Routing\Routes\Compilers\Parsers\Parser;

$dispatcher = new Dispatcher(new Container());
$compiler = new Compiler([new PathMatcher(), new HostMatcher(), new SchemeMatcher()]);
$parser = new Parser();
$router = new Router($dispatcher, $compiler, $parser);
$router->get("/foo", function () {
    return "Hello, world!";
});
```

> **Note:** If you use the <a href="https://github.com/opulencephp/Project" target="_blank">skeleton project</a>, the router is already bound to the IoC container.

If you need any object like the `Request` to be passed into the closure, just type-hint it:

```php
use Opulence\Http\Requests\Request;

$router->get("/users/:id", function (Request $request, $id) {
    // $request will be the HTTP request
    // $id will be the path variable
});
```

<h4 id="using-controller-classes">Using Controller Classes</h4>
Anything other than super-simple applications should probably use full-blown controller classes.  They provide reusability, better separation of responsibilities, and more features.  [Read here](http-basics#controllers) for more information about controllers.

<h4 id="multiple-methods">Multiple Methods</h4>
You can register a route to multiple methods using the router's `multiple()` method:
```php
$router->multiple(["GET", "POST"], "MyApp\\MyController@myMethod");
```

To register a route for all methods, use the `any()` method:
```php
$router->any("MyApp\\MyController@myMethod");
```

<h2 id="route-variables">Route Variables</h2>
Let's say you want to grab a specific user's profile page.  You'll probably want to structure your URL like "/users/:userId/profile", where ":userId" is the Id of the user whose profile we want to view.  Using a `Router`, the data matched in ":userId" will be mapped to a parameter in your controller's method named "$userId".

> **Note:** This also works for closure controllers.  All non-optional parameters in the controller method must have identically-named route variables.  In other words, if your method looks like `function showBook($authorName, $bookTitle = null)`, your path must have an `:authorName` variable.  The routes `/authors/:authorName/books` and `/authors/:authorName/books/:bookTitle` would be valid, but `/authors` would not.

Let's take a look at a full example:
```php
use Opulence\Routing\Controller;

class UserController extends Controller
{
    public function showProfile(int $userId)
    {
        return "Profile for user " . $userId;
    }
}

$router->get("/users/:userId/profile", "MyApp\\UserController@showProfile");
```

Calling the path `/users/23/profile` will return "Profile for user 23".

<h4 id="regular-expressions">Regular Expressions</h4>
If you'd like to enforce certain rules for a route variable, you may do so in the options array.  Simply add a "vars" entry with variable names-to-regular-expression mappings:
```php
$options = [
    "vars" => [
        "userId" => "\d+" // The user Id variable must now be a number
    ]
];
$router->get("/users/:userId/profile", "MyApp\\UserController@showProfile", $options);
```

<h4 id="optional-parts">Optional Parts</h4>
If parts of your route are optional, simply wrap them in `[]`:
```php
$router->get("/books[/authors]", "MyApp\\BookController@showBooks");
```

This would match both `/books` and `/books/authors`.

You can even nest optional parts:

```php
$router->get("/archives[/:year[/:month[/:day]]]", "MyApp\\ArchiveController@showArchives");
```

<h4 id="default-values">Default Values</h4>
Sometimes, you might want to have a default value for a route variable.  Doing so is simple:
```php
$router->get("/food/:foodName=all", "MyApp\\FoodController@showFood");
```

If no food name was specified, "all" will be the default value.

> **Note:** To give an optional variable a default value, structure the route variable like `[:varName=value]`.

<h2 id="host-matching">Host Matching</h2>
Routers can match on hosts as well as paths.  Want to match calls to a subdomain?  Easy:

```php
$options = [
    "host" => "mail.mysite.com" 
];
$router->get("/inbox", "MyApp\\InboxController@showInbox", $options);
```

Just like with paths, you can create variables from components of your host.  In the following example, a variable called `$subdomain` will be passed into `MyApp\SomeController::doSomething()`:

```php
$options = [
    "host" => ":subdomain.mysite.com" 
];
$router->get("/foo", "MyApp\\SomeController@doSomething", $options);
```

Host variables can also have regular expression constraints, similar to path variables.

<h2 id="middleware">Middleware</h2>
Routes can run [middleware](http-middleware) on requests and responses.  To register middleware, add it to the `middleware` property in the route options:

```php
$options = [
    "middleware" => "MyApp\\MyMiddleware" // Can also be an array of middleware
];
$router->get("/books", "MyApp\\MyController@myMethod", $options);
```

Whenever a request matches this route, `MyApp\MyMiddleware` will be run.

<h4 id="middleware-parameters">Middleware Parameters</h4>
Opulence supports [passing primitive parameters to middleware](http-middleware#middleware-parameters).  To actually specify `role`, use `{Your middleware}::withParameters()` in your router configuration:

```php
$router->post("/articles", ["middleware" => RoleMiddleware::withParameters(["role" => "admin"])]);
```

<h2 id="https">HTTPS</h2>
Some routes should only match on an HTTPS connection.  To do this, set the `https` flag to true in the options:

```php
$options = [
    "https" => true
];
$router->get("/users", "MyApp\\MyController@myMethod", $options);
```

HTTPS requests to `/users` will match, but non SSL connections will return a 404 response.

<h2 id="named-routes">Named Routes</h2>
Routes can be given a name, which makes them identifiable.  This is especially useful for things like [generating URLs for a route](#generating-urls-from-code).  To name a route, pass a `"name" => "THE_NAME"` into the route options:

```php
$options = [
    "name" => "awesome"
];
$router->get("/users", "MyApp\\MyController@myMethod", $options);
```

This will create a route named "awesome".

<h2 id="route-grouping">Route Grouping</h2>
One of the most important sayings in programming is "Don't repeat yourself" or "DRY".  In other words, don't copy-and-paste code because that leads to difficulties in maintaining/changing the code base in the future.  Let's say you have several routes that start with the same path.  Instead of having to write out the full path for each route, you can create a group:
```php
$router->group(["path" => "/users/:userId"], function () use ($router) {
    $router->get("/profile", "MyApp\\UserController@showProfile");
    $router->delete("", "MyApp\\UserController@deleteUser");
});
```

Now, a GET request to `/users/:userId/profile` will get a user's profile, and a DELETE request to `/users/:userId` will delete a user.

<h4 id="controller-namespaces">Controller Namespaces</h4>
If all the controllers in a route group belong under a common namespace, you can specify the namespace in the group options:
```php
$router->group(["controllerNamespace" => "MyApp\\Controllers"], function () use ($router) {
    $router->get("/users", "UserController@showAllUsers");
    $router->get("/posts", "PostController@showAllPosts");
});
```

Now, a GET request to `/users` will route to `MyApp\Controllers\UserController::showAllUsers()`, and a GET request to `/posts` will route to `MyApp\Controllers\PostController::showAllPosts()`.

<h4 id="group-middleware">Group Middleware</h4>
Route groups allow you to apply middleware to multiple routes:
```php
$router->group(["middleware" => "MyApp\\Authenticate"], function () use ($router) {
    $router->get("/users/:userId/profile", "MyApp\\UserController@showProfile");
    $router->get("/posts", "MyApp\\PostController@showPosts");
});
```

The `Authenticate` middleware will be executed on any matched routes inside the closure.

<h4 id="group-hosts">Group Hosts</h4>
You can filter by host in router groups:

```php
$router->group(["host" => "google.com"], function () use ($router) {
    $router->get("/", "MyApp\\HomeController@showHomePage");
    $router->group(["host" => "mail."], function () use ($router) {
        $router->get("/", "MyApp\\MailController@showInbox");
    });
});
```

> **Note:** When specifying hosts in nested router groups, the inner groups' hosts are prepended to the outer groups' hosts.  This means the inner-most route in the example above will have a host of "mail.google.com".

<h4 id="group-https">Group HTTPS</h4>
You can force all routes in a group to be HTTPS:

```php
$router->group(["https" => true], function () use ($router) {
    $router->get("/", "MyApp\\HomeController@showHomePage");
    $router->get("/books", "MyApp\\BookController@showBooksPage");
});
```

> **Note:** If the an outer group marks the routes HTTPS but an inner one doesn't, the inner group gets ignored.  The outer-most group with an HTTPS definition is the only one that counts.

<h4 id="group-variable-regular-expressions">Group Variable Regular Expressions</h4>
Groups support regular expressions for path variables:

```php
$options = [
    "path" => "/users/:userId",
    "vars" => [
        "userId" => "\d+"
    ]
];
$router->group($options, function () use ($router) {
    $router->get("/profile", "MyApp\\ProfileController@showProfilePage");
    $router->get("/posts", "MyApp\\PostController@showPostsPage");
});
```

Going to `/users/foo/profile` or `/users/foo/posts` will not match because the Id was not numeric.

> **Note:** If a route has a variable regular expression specified, it takes precedence over group regular expressions.

<h2 id="caching">Caching</h2>
Routes must be parsed to generate the regular expressions used to match the host and path.  This parsing takes a noticeable amount of time with a moderate number of routes.  To make the parsing faster, Opulence caches the parsed routes.  If you're using the <a href="https://github.com/opulencephp/Project" target="_blank">skeleton project</a>, you can enable or disable cache by editing `config/http/routing.php`.

> **Note:** If you're in your production environment, you must run `php apex framework:flushcache` every time you add/modify/delete a route in `config/http/routes.php`.

<h2 id="missing-routes">Missing Routes</h2>
In the case that the router cannot find a route that matches the request, an `Opulence\Http\HttpException` will be thrown with a 404 status code.

<h2 id="url-generators">URL Generators</h2>
A cool feature is the ability to generate URLs from named routes using `Opulence\Routing\Urls\UrlGenerator`.  If your route has variables in the domain or path, you just pass them in `UrlGenerator::createFromName()`.  Unless a host is specified in the route, an absolute path is generated.  Secure routes with hosts specified will generate `https://` absolute URLs.

> **Note:** If you do not define all the non-optional variables in the host or domain, a `UrlException` will be thrown.

<h4 id="generating-urls-from-code">Generating URLs from Code</h4>
```php
use Opulence\Routing\Urls\UrlGenerator;

// Let's assume the router and compiler are already instantiated
$urlGenerator = new UrlGenerator($router->getRoutes(), $compiler);
// Let's add a route named "profile"
$router->get("/users/:userId", "MyApp\\UserController@showProfile", ["name" => "profile"]);
// Now we can generate a URL and pass in data to it
echo $urlGenerator->createFromName("profile", 23); // "/users/23"
```

If we specify a host in our route, an absolute URL is generated.  We can even define variables in the host:

```php
// Let's assume the URL generator is already instantiated
// Let's add a route named "inbox"
$options = [
    "host" => ":country.mail.foo.com",
    "name" => "inbox"
];
$router->get("/users/:userId", "MyApp\\InboxController@showInbox", $options);
// Any values passed in will first be used to define variables in the host
// Any leftover values will define the values in the path
echo $urlGenerator->createFromName("inbox", "us", 2); // "http://us.mail.foo.com/users/2"
```

<h4 id="generating-urls-from-views">Generating URLs from Views</h4>
URLs can also be generated from views using the `route()` view function.  Here's an example router config:

```php
$router->get("/users/:userId/profile", "UserController@showProfile", ["name" => "profile"]);
```

Here's how to generate a URL to the "profile" route:

```php
<a href="{{! route('profile', 123) !}}">View Profile</a>
```

This will compile to:

```
<a href="/users/123/profile">View Profile</a>
```

<h2 id="notes">Notes</h2>
Routes are matched based on the order they were added to the router.  So, if you did the following:
```php
$options = [
    "vars" => [
        "foo" => ".*"
    ]
];
$router->get("/:foo", "MyApp\\MyController@myMethod", $options);
$router->get("/users", "MyApp\\MyController@myMethod");
```

...The first route `/:foo` would always match first because it was added first.  Add any "fall-through" routes after you've added the rest of your routes.