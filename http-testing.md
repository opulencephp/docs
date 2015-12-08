# Testing HTTP Applications

## Table of Contents
1. [Introduction](#introduction)
2. [Testing Routes](#testing-routes)
  1. [Passing Parameters](#passing-parameters)
  2. [JSON](#json)
  3. [with() Methods](#with-methods)
  4. [Mock Requests](#mock-requests)
3. [Assertions](#assertions)
  1. [assertRedirectsTo()](#assert-redirects-to)
  2. [assertResponseContentEquals()](#assert-response-content-equals)
  3. [assertResponseCookieValueEquals()](#assert-response-cookie-value-equals)
  4. [assertResponseHasCookie()](#assert-response-has-cookie)
  5. [assertResponseHasHeader()](#assert-response-has-header)
  6. [assertResponseHeaderEquals()](#assert-response-header-equals)
  7. [assertResponseIsInternalServerError()](#assert-response-is-internal-server-error)
  8. [assertResponseIsNotFound()](#assert-response-is-not-found)
  9. [assertResponseIsOK()](#assert-response-is-ok)
  10. [assertResponseJsonEquals()](#assert-response-json-equals)
  11. [assertResponseStatusCodeEquals()](#assert-response-status-code-equals)
  12. [assertResponseIsUnauthorized()](#assert-response-is-unauthorized)
  13. [assertViewHasTag()](#assert-view-has-tag)
  14. [assertViewHasVar()](#assert-view-has-var)
  15. [assertViewTagEquals()](#assert-view-tag-equals)
  16. [assertViewVarEquals()](#assert-view-var-equals)
4. [Middleware](#middleware)
  1. [Disabling All Middleware](#disabling-all-middleware)
  2. [Disabling Specific Middleware](#disabling-specific-middleware)
  3. [Enabling Specific Middleware](#enabling-specific-middleware)

<h2 id="introduction">Introduction</h2>
Opulence gives you a powerful integration testing tool to simulate routes and test the responses and views created by controllers.  The tool is the `Opulence\Framework\Testing\PhpUnit\Http\ApplicationTestCase` class, which extends PHPUnit's `PHPUnit_Framework_TestCase`.  By extending `ApplicationTestCase`, you'll inherit many methods to help test your application.

> **Note:** If you need to define a `setUp()` or `tearDown()` method in your test, make sure to call `parent::setUp()` or `parent::tearDown()`.

<h2 id="testing-routes">Testing Routes</h2>
Opulence provides a fluent interface to test your routes.  For example, here's how you can make a `POST` request to `/foo` with some data:

```php
public function testPostingSomeData()
{
    $this->post("/foo")
        ->withParameters(["bar" => "baz"])
        ->go()
        ->assertResponseContentEquals("Nice POST request");
}
```

Calling `go()` is strictly optional.  If you don't want to bother and want to get straight to the assertions, you can rewrite it as:

```php
public function testPostingSomeData()
{
    $this->post("/foo")
        ->withParameters(["bar" => "baz"])
        ->assertResponseContentEquals("Nice POST request");
}
```

The following methods create an `Opulence\Framework\Testing\Http\RequestBuilder`, which is useful for creating your requests:

* `delete()`
* `get()`
* `head()`
* `options()`
* `patch()`
* `post()`
* `put()`

<h4 id="passing-parameters">Passing Parameters</h4>
If you're testing a `GET` request and would like to pass some parameters along with it, use `withParameters()`:

```php
public function testWithParameters()
{
    $this->get("/login")
        ->withParameters(["ref" => "http://google.com"])
        ->assertRedirectsTo("http://google.com");
}
```

<h4 id="json">JSON</h4>
To simulate passing JSON data to a route, use `withJson()`.  To assert that a JSON response matches an array, use `assertResponseJsonEquals()`:

```php
public function testJsonRequest()
{
    $this->post("/api/auth")
        ->withJson(["username" => "foo", "password" => "bar"])
        ->assertResponseJsonEquals(["error" => "Invalid username/password"]);
}
```

<h4 id="with-methods">with() Methods</h4>
The following methods can be used to pass data to your request:

* `withCookies($cookies)`
  * Sets the cookies in the request
* `withEnv($env)`
  * Sets the environment vars in the request
* `withFiles($files)`
  * Sets the files in the request
  * `$files` must be a list of `Opulence\Http\Requests\UploadedFile` objects
* `withHeaders($headers)`
  * Sets the headers in the request
  * `HTTP_` is automatically prepended to the header names
* `withJson($json)`
  * Sets the request body to the input JSON
* `withParameters($parameters)`
  * Sets the request query to `$parameters` on `GET` requests, and sets the request post to `$parameters` on `POST` requests
* `withRawBody($rawBody)`
  * Sets the raw body in the request
* `withServerVars($serverVars)`
  * Sets the server vars in the request

<h4 id="mock-requests">Mock Requests</h4>
If you want fine-grained control over your tests, you can pass in a `Request` object to `ApplicationTestCase::route()`:

```php
public function testPostingSomeData()
{
    $request = new Request(["name" => "Dave"], [], [], [], [], []);
    $this->route($request);
    $this->assertResponseContentEquals("Hello, Dave");
}
```

> **Note:**  If you're not using a `RequestBuilder`, you must call `route()` before any assertions defined in `ApplicationTestCase`.

<h2 id="assertions">Assertions</h2>

<h2 id="assert-redirects-to">assertRedirectsTo()</h2>
Asserts that the response is a redirect to a particular URL:

```php
public function testMyRedirect()
{
    $this->get("/myaccount");
    $this->assertRedirectsTo("/login");
}
```

<h4 id="assert-response-content-equals">assertResponseContentEquals()</h4>
Asserts that the response's content matches an expected value:

```php
public function testContent()
{
    $this->get("/404");
    $this->assertResponseContentEquals("Page not found");
}
```

<h4 id="assert-response-cookie-value-equals">assertResponseCookieValueEquals()</h4>
Asserts that a response cookie matches an expected value:

```php
public function testCheckingVisitedCookie()
{
    $this->get("/home");
    $this->assertResponseCookieValueEquals("visited", "1");
}
```

<h4 id="assert-response-has-cookie">assertResponseHasCookie()</h4>
Asserts that a response has a cookie with a particular name:

```php
public function testCheckingVisitedCookie()
{
    $this->get("/home");
    $this->assertResponseHasCookie("visited");
}
```

<h4 id="assert-response-has-header">assertResponseHasHeader()</h4>
Asserts that a response has a header with a particular name:

```php
public function testCheckingCacheControl()
{
    $this->get("/profile");
    $this->assertResponseHasHeader("cache-control");
}
```

<h4 id="assert-response-header-equals">assertResponseHeaderEquals()</h4>
Asserts that a response header matches an expected value:

```php
public function testCheckingCacheControl()
{
    $this->get("/profile");
    $this->assertResponseHeaderEquals("cache-control", "no-cache, must-revalidate");
}
```

<h4 id="assert-response-is-internal-server-error">assertResponseIsInternalServerError()</h4>
Asserts that a response is an internal server error:

```php
public function testBadRequest()
{
    $this->get("/500");
    $this->assertResponseIsInternalServerError();
}
```

<h4 id="assert-response-is-not-found">assertResponseIsNotFound()</h4>
Asserts that a response is not found:

```php
public function test404()
{
    $this->get("/404");
    $this->assertResponseIsNotFound();
}
```

<h4 id="assert-response-is-ok">assertResponseIsOK()</h4>
Asserts that a response is OK:

```php
public function testHomepage()
{
    $this->get("/");
    $this->assertResponseIsOK();
}
```

<h4 id="assert-response-json-equals">assertResponseJsonEquals()</h4>
Asserts that a JSON response matches an input array when decoded:

```php
public function testJsonResponse()
{
    $this->get("/api/auth")
        ->assertResponseJsonEquals(["foo" => "bar"]);
}
```

<h4 id="assert-response-status-code-equals">assertResponseStatusCodeEquals()</h4>
Asserts that a response's status code equals a particular value:

```php
public function testPaymentRequiredOnSubscriptionPage()
{
    $this->get("/subscribers/reports");
    $this->assertResponseStatusCodeEquals(ResponseHeaders::HTTP_PAYMENT_REQUIRED);
}
```

<h4 id="assert-response-is-unauthorized">assertResponseIsUnauthorized()</h4>
Asserts that a response is not authorized:

```php
public function testUserListPageWhenNotLoggedIn()
{
    $this->get("/users");
    $this->assertResponseIsUnauthorized();
}
```

<h4 id="assert-view-has-tag">assertViewHasTag()</h4>
Asserts that the view generated by the controller has a particular tag:

```php
public function testTitleIsSet()
{
    $this->get("/home");
    $this->assertViewHasTag("title");
}
```

> **Note:** This is only available if your controller extends `Opulence\Routing\Controller`.

<h4 id="assert-view-has-var">assertViewHasVar()</h4>
Asserts that the view generated by the controller has a particular variable:

```php
public function testCSSVariableIsSet()
{
    $this->get("/home");
    $this->assertViewHasVar("css");
}
```

> **Note:** This is only available if your controller extends `Opulence\Routing\Controller`.

<h4 id="assert-view-tag-equals">assertViewTagEquals()</h4>
Asserts that the view generated by the controller has a particular tag with an expected value:

```php
public function testTitleIsSet()
{
    $this->get("/home");
    $this->assertViewTagEquals("title", "Home");
}
```

> **Note:** This is only available if your controller extends `Opulence\Routing\Controller`.

<h4 id="assert-view-var-equals">assertViewVarEquals()</h4>
Asserts that the view generated by the controller has a particular variable with an expected value:

```php
public function testCSSVariableIsSet()
{
    $this->get("/home");
    $this->assertViewVarEquals("css", ["assets/css/style.css"]);
}
```

> **Note:** This is only available if your controller extends `Opulence\Routing\Controller`.

<h2 id="middleware">Middleware</h2>
You can customize which middleware are run in your tests.

<h4 id="disabling-all-middleware">Disabling All Middleware</h4>
You may disable all middleware in your kernel:

```php
public function testWithoutMiddleware()
{
    $this->getKernel()->disableAllMiddleware();
    // ...Do your tests
}
```

<h4 id="disabling-specific-middleware">Disabling Specific Middleware</h4>
You may disable only specific middleware in your kernel:

```php
public function testWithoutSpecificMiddleware()
{
    $this->getKernel()->onlyDisableMiddleware(["MyMiddlewareClass"]);
    // ...Do your tests
}
```

<h4 id="enabling-specific-middleware">Enabling Specific Middleware</h4>
You may enable only specific middleware in your kernel:

```php
public function testWithSpecificMiddleware()
{
    $this->getKernel()->onlyEnableMiddleware(["MyMiddlewareClass"]);
    // ...Do your tests
}
```