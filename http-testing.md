# Testing HTTP Applications

## Table of Contents
1. [Introduction](#introduction)
2. [Testing Routes](#testing-routes)
  1. [Passing Parameters](#passing-parameters)
  2. [JSON](#json)
  3. [with() Methods](#with-methods)
  4. [Mock Requests](#mock-requests)
3. [Response Assertions](#response-assertions)
  1. [contentEquals()](#assert-response-content-equals)
  2. [cookieValueEquals()](#assert-response-cookie-value-equals)
  3. [hasCookie()](#assert-response-has-cookie)
  4. [hasHeader()](#assert-response-has-header)
  5. [headerEquals()](#assert-response-header-equals)
  6. [isInternalServerError()](#assert-response-is-internal-server-error)
  7. [isNotFound()](#assert-response-is-not-found)
  8. [isOK()](#assert-response-is-ok)
  9. [isUnauthorized()](#assert-response-is-unauthorized)
  10. [jsonContains()](#assert-response-json-contains)
  11. [jsonContainsKey()](#assert-response-json-contains)
  12. [jsonEquals()](#assert-response-json-equals)
  13. [redirectsTo()](#assert-redirects-to)
  14. [statusCodeEquals()](#assert-response-status-code-equals)
4. [View Assertions](#view-assertions)
  1. [hasVar()](#assert-view-has-var)
  2. [varEquals()](#assert-view-var-equals)
4. [Middleware](#middleware)
  1. [Disabling All Middleware](#disabling-all-middleware)
  2. [Disabling Specific Middleware](#disabling-specific-middleware)
  3. [Enabling Specific Middleware](#enabling-specific-middleware)

<h2 id="introduction">Introduction</h2>
Opulence gives you a powerful integration testing tool to simulate routes and test the responses and views created by controllers.  The tool is the `Opulence\Framework\Http\Testing\PhpUnit\IntegrationTestCase` class, which extends PHPUnit's `PHPUnit_Framework_TestCase`.  By extending `IntegrationTestCase`, you'll inherit many methods to help test your application.

> **Note:** If you need to define a `setUp()` or `tearDown()` method in your test, make sure to call `parent::setUp()` or `parent::tearDown()`.

<h2 id="testing-routes">Testing Routes</h2>
Opulence provides a fluent interface to test your routes.  For example, here's how you can make a `POST` request to `/foo` with some data:

```php
public function testPostingSomeData()
{
    $this->post('/foo')
        ->withParameters(['bar' => 'baz'])
        ->go()
        ->assertResponse
        ->contentEquals('Nice POST request');
}
```

The following methods create an `Opulence\Framework\Http\Testing\PhpUnit\RequestBuilder`, which is useful for creating your requests:

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
    $this->get('/login')
        ->withParameters(['ref' => 'http://google.com'])
        ->go()
        ->assertResponse
        ->redirectsTo('http://google.com');
}
```

<h4 id="json">JSON</h4>
To simulate passing JSON data to a route, use `withJson()`.  To assert that a JSON response matches an array, use `jsonEquals()`:

```php
public function testJsonRequest()
{
    $this->post('/api/auth')
        ->withJson(['username' => 'foo', 'password' => 'bar'])
        ->go()
        ->assertResponse
        ->jsonEquals(['error' => 'Invalid username/password']);
}
```

<h4 id="with-methods">with() Methods</h4>
The following methods can be used to pass data to your request:

* `withCookies($cookies)`
  * Sets the cookies in the request
* `withEnvironmentVars($env)`
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
If you want fine-grained control over your tests, you can pass in a `Request` object to `IntegrationTestCase::route()`:

```php
public function testPostingSomeData()
{
    $request = new Request(['name' => 'Dave'], [], [], [], [], []);
    $this->route($request);
    $this->assertResponse
        ->contentEquals('Hello, Dave');
}
```

> **Note:**  If you're not using a `RequestBuilder`, you must call `route()` before any assertions defined in `IntegrationTestCase`.

<h2 id="response-assertions">Response Assertions</h2>
You can run various assertions on the response returned by the `Kernel`.  To do so, simply use `$this->assertResponse`.

<h4 id="assert-response-content-equals">contentEquals()</h4>
Asserts that the response's content matches an expected value:

```php
public function testContent()
{
    $this->get('/404')
        ->go()
        ->assertResponse
        ->contentEquals('Page not found');
}
```

<h4 id="assert-response-cookie-value-equals">cookieValueEquals()</h4>
Asserts that a response cookie matches an expected value:

```php
public function testCheckingVisitedCookie()
{
    $this->get('/home')
        ->go()
        ->assertResponse
        ->cookieValueEquals('visited', '1');
}
```

<h4 id="assert-response-has-cookie">hasCookie()</h4>
Asserts that a response has a cookie with a particular name:

```php
public function testCheckingVisitedCookie()
{
    $this->get('/home')
        ->go()
        ->assertResponse
        ->hasCookie('visited');
}
```

<h4 id="assert-response-has-header">hasHeader()</h4>
Asserts that a response has a header with a particular name:

```php
public function testCheckingCacheControl()
{
    $this->get('/profile')
        ->go()
        ->assertResponse
        ->hasHeader('cache-control');
}
```

<h4 id="assert-response-header-equals">headerEquals()</h4>
Asserts that a response header matches an expected value:

```php
public function testCheckingCacheControl()
{
    $this->get('/profile')
        ->go()
        ->assertResponse
        ->headerEquals('cache-control', 'no-cache, must-revalidate');
}
```

<h4 id="assert-response-is-internal-server-error">isInternalServerError()</h4>
Asserts that a response is an internal server error:

```php
public function testBadRequest()
{
    $this->get('/500')
        ->go()
        ->assertResponse
        ->isInternalServerError();
}
```

<h4 id="assert-response-is-not-found">isNotFound()</h4>
Asserts that a response is not found:

```php
public function test404()
{
    $this->get('/404')
        ->go()
        ->assertResponse
        ->isNotFound();
}
```

<h4 id="assert-response-is-ok">isOK()</h4>
Asserts that a response is OK:

```php
public function testHomepage()
{
    $this->get('/')
        ->go()
        ->assertResponse
        ->isOK();
}
```

<h4 id="assert-response-is-unauthorized">isUnauthorized()</h4>
Asserts that a response is not authorized:

```php
public function testUserListPageWhenNotLoggedIn()
{
    $this->get('/users')
        ->go()
        ->assertResponse
        ->isUnauthorized();
}
```

<h4 id="assert-response-json-contains">jsonContains()</h4>
Asserts that a JSON response contains the key/value pairs anywhere in the response.  This does not require strict equality like `jsonEquals()`.

```php
public function testJsonResponse()
{
    $this->get('/user/123')
        ->go()
        ->assertResponse
        ->jsonContains(['name' => 'Dave']);
}
```

<h4 id="assert-response-json-contains-key">jsonContainsKey()</h4>
Asserts that a JSON response contains the key anywhere in the response.

```php
public function testJsonResponse()
{
    $this->get('/user/123')
        ->go()
        ->assertResponse
        ->jsonContainsKey('name');
}
```

<h4 id="assert-response-json-equals">jsonEquals()</h4>
Asserts that a JSON response matches an input array when decoded:

```php
public function testJsonResponse()
{
    $this->get('/api/auth')
        ->go()
        ->assertResponse
        ->jsonEquals(['foo' => 'bar']);
}
```

<h4 id="assert-redirects-to">redirectsTo()</h4>
Asserts that the response is a redirect to a particular URL:

```php
public function testMyRedirect()
{
    $this->get('/myaccount')
        ->go()
        ->assertResponse
        ->redirectsTo('/login');
}
```

<h4 id="assert-response-status-code-equals">statusCodeEquals()</h4>
Asserts that a response's status code equals a particular value:

```php
public function testPaymentRequiredOnSubscriptionPage()
{
    $this->get('/subscribers/reports')
        ->go()
        ->assertResponse
        ->statusCodeEquals(ResponseHeaders::HTTP_PAYMENT_REQUIRED);
}
```

<h2 id="view-assertions">View Assertions</h2>
If your controller extends `Opulence\Routing\Controller`, you can test the [view](view-basics) set in the response using `$this->assertView`.

<h4 id="assert-view-has-var">hasVar()</h4>
Asserts that the view generated by the controller has a particular variable:

```php
public function testCssVariableIsSet()
{
    $this->get('/home')
        ->go()
        ->assertView
        ->hasVar('css');
}
```

<h4 id="assert-view-var-equals">varEquals()</h4>
Asserts that the view generated by the controller has a particular variable with an expected value:

```php
public function testCssVariableIsSet()
{
    $this->get('/home')
        ->go()
        ->assertView
        ->varEquals('css', ['assets/css/style.css']);
}
```

<h2 id="middleware">Middleware</h2>
You can customize which middleware are run in your tests.

<h4 id="disabling-all-middleware">Disabling All Middleware</h4>
You may disable all middleware in your kernel:

```php
public function testWithoutMiddleware()
{
    $this->kernel->disableAllMiddleware();
    // ...Do your tests
}
```

<h4 id="disabling-specific-middleware">Disabling Specific Middleware</h4>
You may disable only specific middleware in your kernel:

```php
public function testWithoutSpecificMiddleware()
{
    $this->kernel->onlyDisableMiddleware(['MyMiddlewareClass']);
    // ...Do your tests
}
```

<h4 id="enabling-specific-middleware">Enabling Specific Middleware</h4>
You may enable only specific middleware in your kernel:

```php
public function testWithSpecificMiddleware()
{
    $this->kernel->onlyEnableMiddleware(['MyMiddlewareClass']);
    // ...Do your tests
}
```
