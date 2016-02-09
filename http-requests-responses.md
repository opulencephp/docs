# HTTP Requests/Responses

## Table of Contents
1. [Introduction](#introduction)
2. [Requests](#requests)
  1. [Collections](#collections)
  2. [Query String Data](#query-string-data)
  3. [Post Data](#post-data)
  4. [Request Data](#request-data)
  5. [Put Data](#put-data)
  6. [Patch Data](#patch-data)
  7. [Delete Data](#delete-data)
  8. [Cookies](#cookies)
  9. [Server Data](#server-data)
  10. [Header Data](#header-data)
  11. [File Data](#file-data)
  12. [Env Data](#env-data)
  13. [Getting the Path](#getting-the-path)
      1. [Checking the Path](#checking-the-path)
  14. [Getting the Method](#getting-the-method)
  15. [Body](#body)
      1. [JSON](#json)
      2. [Raw Body](#raw-body)
  16. [AJAX](#ajax)
  17. [Getting the Client IP Address](#getting-the-client-ip-address)
  18. [Checking if HTTPS](#checking-if-https)
  19. [Getting the Full URL](#getting-the-full-url)
      1. [Checking the Full URL](#checking-the-full-url)
  20. [Getting the Previous URL](#getting-the-previous-url)
  21. [Authentication Data](#authentication-data)
  22. [Using Proxies](#using-proxies)
3. [Responses](#responses)
  1. [Status Codes](#status-codes)
  2. [Headers](#headers)
  3. [Cookies](#cookies)
      1. [Setting Cookies](#setting-cookies)
      2. [Deleting Cookies](#deleting-cookies)
  4. [JSON Responses](#json-responses)
  5. [Redirect Responses](#redirect-responses)
  6. [Stream Responses](#stream-responses)

<h2 id="introduction">Introduction</h2>
Opulence makes interacting with HTTP requests and responses easy.  Tasks like checking if a POST variable is set before using it are repetitive when working directly with PHP's `$_POST` global array.  If you've ever worked with cookies and gotten the "headers already sent" error, you know how annoying it is to work with the HTTP tools PHP gives you by default.  Use Opulence's tools, and stop worry about stuff like this.
  
<h2 id="requests">Requests</h2>
Opulence has a wrapper around an HTTP request in the `Opulence\Http\Requests\Request` class.

<h4 id="collections">Collections</h4>
Superglobal request data, eg `$_GET` and `$_POST` are wrapped into `Opulence\Http\Collection` objects, which have the following methods:

* `get($key)`
* `has($key)`
* `remove($key)`
* `set($key, $value)`

<h4 id="query-string-data">Query String Data</h4>
```php
$request->getQuery()->get("foo");
```

> **Note:** This is similar to the data in `$_GET`.  `getQuery()` returns a `Collection` object.

<h4 id="post-data">Post Data</h4>
```php
$request->getPost()->get("foo");
```

> **Note:** This is similar to the data in `$_POST`.  `getPost()` returns a `Collection` object.

<h4 id="request-data">Request Data</h4>
```php
$request->getInput("foo");
```

> **Note:** This is similar to the data in `$_REQUEST`.  This is useful if you do not know if form data is coming from the query string, post data, delete data, put data, or patch data.  If the request body was JSON, `getInput()` returns the JSON property with the input name.

<h4 id="put-data">Put Data</h4>
```php
$request->getPut()->get("foo");
```

> **Note:** This is only populated with `x-www-form-urlencoded` content type.  `getPut()` returns a `Collection` object.

<h4 id="patch-data">Patch Data</h4>
```php
$request->getPatch()->get("foo");
```

> **Note:** This is only populated with `x-www-form-urlencoded` content type.  `getPatch()` returns a `Collection` object.

<h4 id="delete-data">Delete Data</h4>
```php
$request->getDelete()->get("foo");
```

> **Note:** This is only populated with `x-www-form-urlencoded` content type.  `getDelete()` returns a `Collection` object.

<h4 id="cookies">Cookies</h4>
```php
$request->getCookies()->get("foo");
```

> **Note:** This is similar to the data in `$_COOKIE`.  `getCookies()` returns a `Collection` object.

<h4 id="server-data">Server Data</h4>
```php
$request->getServer()->get("foo");
```

> **Note:** This is similar to the data in `$_SERVER`.  `getServer()` returns a `Collection` object.

<h4 id="header-data">Header Data</h4>
```php
$request->getHeaders()->get("foo");
```

> **Note:** These are the $\_SERVER values whose names with with "HTTP\_".  `getHeaders()` returns a `Collection` object.  Their names are case-insensitive per RFC 2616.

<h4 id="file-data">File Data</h4>
```php
$request->getFiles()->get("foo");
```

> **Note:** This is similar to the data in `$_FILES`.  `getFiles()` returns a `Files` object.

Opulence converts `$_FILES` data into an array of `Opulence\Http\Requests\UploadedFile` objects.  This provides an easy-to-use wrapper around PHP's built-in upload capabilities.  Each `UploadedFile` object extends `\SplFileInfo` and adds the following methods:

```php
// The error code for the uploaded file (UPLOAD_ERR_OK indicates no error)
$uploadedFile->getError();
// The actual mime type of the uploaded file (secure)
$uploadedFile->getMimeType();
// The extension claimed by the uploaded file (not secure)
$uploadedFile->getTempExtension();
// The mime type claimed by the uploaded file (not secure)
$uploadedFile->getTempMimeType();
// The temporary filename of the uploaded file
$uploadedFile->getTempFilename();
// Whether or not the uploaded file has any errors besides UPLOAD_ERR_OK
$uploadedFile->hasErrors();
// Moves the temporary file to the specified directory/name
$uploadedFile->move($targetDirectory, $name);
```

<h4 id="env-data">Env Data</h4>
```php
$request->getEnv()->get("foo");
```

> **Note:** This is similar to the data in `$_ENV`.  `getEnv()` returns a `Collection` object.

<h4 id="getting-the-path">Getting the Path</h4>
```php
$request->getPath();
```
  
<h5 id="checking-the-path">Checking the Path</h5>
Opulence allows you to check if a certain path is the current path using `$request->isPath(PATH_TO_MATCH)`.  It also allows you to use a regular expression for more complex matching:

```php
// The second parameter lets the request know that we're using a regular expression
$request->isPath("/docs/.*", true);
```

> **Note:** Do not include regular expression delimiters in your regular expression.

<h4 id="getting-the-method">Getting the Method</h4>
To determine which HTTP method was used to make a request (eg "GET" or "POST"), use `getMethod()`:
```php
$request->getMethod();
```

<h5 id="spoofing-http-methods">Spoofing HTTP Methods</h5>
HTML forms only permit `GET` and `POST` HTTP methods.  You can spoof other methods by including a hidden input named "_method" whose value is the method you'd like to spoof:

```php
<form method="POST">
    <input type="hidden" name="_method" value="DELETE">
    ...
</form>
```

If you are using Fortune for your views, you may use the `httpMethodInput()` [view function](view-fortune#functions):

```php
<form method="POST">
    {{! httpMethodInput("DELETE") !}}
    ...
</form>
```

You can also use the `X-HTTP-METHOD-OVERRIDE` header to set the HTTP method.
  
<h4 id="body">Body</h4>
The body of a request comes from the `php://input` stream.  It can be used to grab non-form data, such as JSON and XML data.

<h5 id="json">JSON</h5>
Opulence can accept JSON as input, which makes it easy to start developing REST applications:

```php
$request->getJsonBody();
```

You can check if a request is JSON:

```php
$request->isJson();
```

<h5 id="raw-body">Raw Body</h5>
If you want access to the raw request body, run:

```php
$request->getRawBody();
```

<h4 id="ajax">AJAX</h4>
To determine if a request was made by AJAX, call:

```php
$request->isAjax();
```

<h4 id="getting-the-client-ip-address">Getting the Client IP Address</h4>
```php
$request->getClientIPAddress();
```

<h4 id="checking-if-https">Checking if HTTPS</h4>
```php
$request->isSecure();
```

<h4 id="getting-the-full-url">Getting the Full URL</h4>
```php
$request->getFullUrl();
```

<h5 id="checking-the-full-url">Checking the Full URL</h5>
Opulence allows you to check if a certain URL is the current URL using `$request->isUrl(URL_TO_MATCH)`.  It also allows you to use a regular expression for more complex matching:

```php
// The second parameter lets the request know that we're using a regular expression
$request->isUrl("http://mysite\.com/posts/.*", true);
```

> **Note:** Do not include regular expression delimiters in your regular expression.

<h4 id="getting-the-previous-url">Getting the Previous URL</h4>
```php
$request->getPreviousUrl();
```
> **Note:** This only works when using the session middleware.

<h4 id="authentication-data">Authentication Data</h4>
```php
$phpAuthUser = $request->getUser();
$phpAuthPassword = $request->getPassword();
```

<h4 id="using-proxies">Using Proxies</h4>
If you're using a load balancer or some sort of proxy server, you'll need to add it to the list of trusted proxies:

```php
Request::setTrustedProxies(["192.168.128.41"]);
```

If you want to use your proxy to set trusted headers, then specify their names:

```php
use Opulence\Http\Requests\RequestHeaders;

Request::setTrustedHeaderName(RequestHeaders::CLIENT_IP, "X-Proxy-Ip");
Request::setTrustedHeaderName(RequestHeaders::CLIENT_HOST, "X-Proxy-Host");
Request::setTrustedHeaderName(RequestHeaders::CLIENT_PORT, "X-Proxy-Port");
Request::setTrustedHeaderName(RequestHeaders::CLIENT_PROTO, "X-Proxy-Proto");
```

> **Note:** If you're using the <a href="https://github.com/opulencephp/Project" target="_blank">skeleton project</a>, then a [bootstrapper](bootstrappers) would be a good place to set the proxy settings.

<h2 id="responses">Responses</h2>
Opulence also wraps an HTTP response into the `Opulence\Http\Responses\Response` class.  You can set the content, HTTP status code, headers, and cookies in a response.  Opulence also supports JSON responses and redirects.

<h4 id="status-codes">Status Codes</h4>
By default, a status code of 200 (HTTP OK) is returned.  A full list of codes is available in `Opulence\Http\Responses\ResponseHeaders`.  For example, here's how to set an "Unauthorized" status code:

```php
use Opulence\Http\Responses\Response;
use Opulence\Http\Responses\ResponseHeaders;

$response = new Response("Permission denied", ResponseHeaders::HTTP_UNAUTHORIZED);
$response->send();
```

<h4 id="headers">Headers</h4>
You can specify any headers you'd like in your response.  To do something like set a "Content-Type" header to an octet stream, do the following:

```php
use Opulence\Http\Responses\Response;
use Opulence\Http\Responses\ResponseHeaders;

$response = new Response("Foo");
$response->getHeaders()->set("Content-Type", ResponseHeaders::CONTENT_TYPE_OCTET_STREAM);
```

<h4 id="cookies">Cookies</h4>
<h5 id="setting-cookies">Setting Cookies</h5>
Cookies are meant to help you remember information about a user, such as authentication credentials or site preferences.  Opulence wraps a cookie into the `Opulence\Http\Responses\Cookie` class.  To create a cookie, first create a `Cookie` object.  Then, add it to the `Response` object's headers so that it will be sent to the user.

```php
use DateTime;
use Opulence\Http\Responses\Cookie;

// This should look pretty familiar to PHP's setcookie() function
$userIdCookie = new Cookie("id", 17, new DateTime("+1 week"), "/", ".foo.com", true, true);
$response->getHeaders()->setCookie($userIdCookie);
$response->send();
```

> **Note:** Unlike PHP's `setcookie()`, Opulence defaults to setting the `httpOnly` flag to true.  This makes sites less prone to cross-site request forgeries. 

<h5 id="deleting-cookies">Deleting Cookies</h5>
Deleting a cookie is easy:

```php
// Let's delete the cookie we set in the example above
$response->getHeaders()->deleteCookie("id", "/", ".foo.com", true, true);
$response->send();
```

<h4 id="json-responses">JSON Responses</h4>
JSON responses are great for API calls.  Opulence supports JSON responses in the `Opulence\Http\Responses\JsonResponse` class.  It accepts either an `array` or `ArrayObject` as its content.

```php
use Opulence\Http\Responses\JsonResponse;
use Opulence\Http\Responses\ResponseHeaders;

$response = new JsonResponse(["message" => "Hello, world!"], ResponseHeaders::HTTP_OK);
$response->send();
```

This will yield the following JSON:

```javascript
{"message": "Hello, world!"}
```

<h4 id="redirect-responses">Redirect Responses</h4>
You'll often finding yourself wanting to redirect a user after things like logging in or out or after filling out a form.  Use an `Opulence\Http\Responses\RedirectResponse`:

```php
use Opulence\Http\Responses\RedirectResponse;
use Opulence\Http\Responses\ResponseHeaders;

$response = new RedirectResponse("http://www.example.com/login", ResponseHeaders::HTTP_FOUND);
$response->send();
```

<h4 id="stream-responses">Stream Responses</h4>
If your response should be sent as a stream, use `Opulence\Http\Responses\StreamResponse`.  Simply pass a callback that outputs the contents:

```php
use Opulence\Http\Responses\StreamResponse;

$response = new StreamResponse(function () {
    echo "My awesome stream";
    // To make sure the output gets sent, flush it
    flush();
});
```

> **Note:** `StreamResponse` does not let you use the method `setContent()`.  Instead, pass in the callback in the constructor or in the `setStreamCallback()` method.