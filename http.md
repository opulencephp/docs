# HTTP Requests/Responses

## Table of Contents
1. [Introduction](#introduction)
2. [Requests](#requests)
  1. [Body](#body)
    1. [JSON](#json)
    2. [Raw Body](#raw-body)
  2. [AJAX](#ajax)
  3. [Checking Path](#checking-path)
3. [Responses](#responses)
  1. [Status Codes](#status-codes)
  2. [Headers](#headers)
  3. [Cookies](#cookies)
    1. [Setting Cookies](#setting-cookies)
    2. [Deleting Cookies](#deleting-cookies)
  4. [JSON Responses](#json-responses)
  5. [Redirect Responses](#redirect-responses)

<h2 id="introduction">Introduction</h2>
RDev makes interacting with HTTP requests and responses easy.  Tasks like checking if a POST variable is set before using it are repetitive when working directly with PHP's `$_POST` global array.  If you've ever worked with cookies and gotten the "headers already sent" error, you know how annoying it is to work with the HTTP tools PHP gives you by default.  Use RDev's tools, and stop worry about stuff like this.
  
<h2 id="requests">Requests</h2>
RDev has a wrapper around an HTTP request in the `RDev\HTTP\Requests\Request` class.  This includes storing the following pieces of data:
* $_GET
  * Available through `$request->getQuery()`
* $_POST
  * Available through `$request->getPost()`
* PUT data
  * Available through `request->getPut()`
  * Only populated with `x-www-form-urlencoded` content type
* PATCH data
  * Available through `request->getPatch()`
  * Only populated with `x-www-form-urlencoded` content type
* DELETE data
  * Available through `request->getDelete()`
  * Only populated with `x-www-form-urlencoded` content type
* $_COOKIE
  * Available through `$request->getCookies()`
* $_SERVER
  * Available through `$request->getServer()`
* HTTP headers
  * Available through `$request->getHeaders()`
  * These are the $\_SERVER values whose names began with "HTTP\_"
* $_FILES
  * Available through `$request->getFiles()`
* $_ENV
  * Available through `$request->getEnv()`
* IP address
  * Available through `$request->getIPAddress()`
  * Automatically handles HTTP forwarded IP addresses
* HTTPS
  * Available through `$request->isSecure()`
* Path
  * Available through `$request->getPath()`
  
<h4 id="body">Body</h4>
The body of a request comes from the `php://input` stream.  It can be used to grab non-form data, such as JSON and XML data.

<h5 id="json">JSON Body</h5>
RDev can accept JSON as input, which makes it easy to start developing REST applications:

```php
$request->getJSONBody();
```

<h5 id="raw-input">Raw Body</h5>
If you want access to the raw request body, run:

```php
$request->getRawBody();
```

<h4 id="ajax">AJAX</h4>
To determine if a request was made by AJAX, call:

```php
$request->isAJAX();
```
  
<h4 id="checking-path">Checking Path</h4>
RDev allows you to check if a certain path is the current path using `$request->isPath(PATH_TO_MATCH)`.  It also allows you to use a regular expression for more complex matching:

```php
// The second parameter lets the request know that we're using a regular expression
$request->isPath("/docs/.*", true);
```

> **Note:** Do not include regular expression delimiters in your regular expression.

<h2 id="responses">Responses</h2>
RDev also wraps an HTTP response into the `RDev\HTTP\Responses\Response` class.  You can set the content, HTTP status code, headers, and cookies in a response.  RDev also supports JSON responses and redirects.

<h4 id="status-codes">Status Codes</h4>
By default, a status code of 200 (HTTP OK) is returned.  A full list of codes is available in `RDev\HTTP\Responses\ResponseHeaders`.  For example, here's how to set an "Unauthorized" status code:

```php
use RDev\HTTP\Responses;

$response = new Responses\Response("Permission denied", Responses\ResponseHeaders::HTTP_UNAUTHORIZED);
$response->send();
```

<h4 id="headers">Headers</h4>
You can specify any headers you'd like in your response.  To do something like set a "Content-Type" header to an octet stream, do the following:

```php
use RDev\HTTP\Responses;

$response = new Responses\Response("Foo");
$response->getHeaders()->set("Content-Type", Responses\ResponseHeaders::CONTENT_TYPE_OCTET_STREAM);
```

<h4 id="cookies">Cookies</h4>
<h5 id="setting-cookies">Setting Cookies</h5>
Cookies are meant to help you remember information about a user, such as authentication credentials or site preferences.  RDev wraps a cookie into the `RDev\HTTP\Responses\Cookie` class.  To create a cookie, first create a `Cookie` object.  Then, add it to the `Response` object's headers so that it will be sent to the user.

```php
use RDev\HTTP\Responses;

// This should look pretty familiar to PHP's setcookie() function
$userIdCookie = new Responses\Cookie("userId", 12345, new \DateTime("+1 week"), "/", ".example.com", true, true);
$response->getHeaders()->setCookie($userIdCookie);
$response->send();
```

> **Note:** Unlike PHP's `setcookie()`, RDev defaults to setting the `httpOnly` flag to true.  This makes sites less prone to cross-site request forgeries. 

<h5 id="deleting-cookies">Deleting Cookies</h5>
Deleting a cookie is easy:

```php
use RDev\HTTP\Responses;

// Let's delete the cookie we set in the example above
$response->getHeaders()->deleteCookie("userId", "/", ".example.com", true, true);
$response->send();
```

<h4 id="json-responses">JSON Responses</h4>
JSON responses are great for API calls.  RDev supports JSON responses in the `RDev\HTTP\Responses\JSONResponse` class.  It accepts either an `array` or `ArrayObject` as its content.

```php
use RDev\HTTP\Responses;

$response = new Responses\JSONResponse(["message" => "Hello, world!"], Responses\ResponseHeaders::HTTP_OK);
$response->send();
```

This will yield the following JSON:

```javascript
{"message": "Hello, world!"}
```

<h4 id="redirect-responses">Redirect Responses</h4>
You'll often finding yourself wanting to redirect a user after things like logging in or out or after filling out a form.  Use an `RDev\HTTP\Responses\RedirectResponse`:

```php
use RDev\HTTP\Responses;

$response = new Responses\RedirectResponse("http://www.example.com/login", Responses\ResponseHeaders::HTTP_FOUND);
$response->send();
```