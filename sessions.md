# Sessions

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Usage](#basic-usage)
  1. [Setting Data](#setting-data)
  2. [Getting Data](#getting-data)
  3. [Getting All Data](#getting-all-data)
  4. [Checking if a Session Has a Key](#checking-if-session-has-key)
  5. [Deleting Data](#deleting-data)
  6. [Flushing All Data](#flushing-all-data)
  7. [Flashing Data](#flashing-data)
  8. [Regenerating the Id](#regenerating-the-id)
3. [Session Handlers](#session-handlers)
4. [Middleware](#middleware)
5. [Using Sessions In Controllers](#using-sessions-in-controllers)
6. [Id Generators](#id-generators)
7. [Encrypting Session Data](#encrypting-session-data)
8. [Configuration](#configuration)

<h2 id="introduction">Introduction</h2>
HTTP is a stateless protocol.  What that means is that each request has no memory of previous requests.  If you've ever used the web, though, you've probably noticed that websites are able to remember information across requests.  For example, a "shopping cart" on an e-commerce website remembers what items you've added to your cart.  How'd they do that?  **Sessions**.

> **Note:** Although similar in concept, Opulence's sessions do not use PHP's built-in `$_SESSION` functionality because it is awful.

<h2 id="basic-usage">Basic Usage</h2>
Opulence sessions must implement `Opulence\Sessions\ISession` (`Opulence\Sessions\Session` comes built-in).

<h4 id="setting-data">Setting Data</h4>
Any kind of serializable data can be written to sessions:

```php
use Opulence\Sessions\Session;

$session = new Session();
$session->set('someString', 'foo');
$session->set('someArray', ['bar', 'baz']);
```

<h4 id="getting-data">Getting Data</h4>
```php
$session->set('someKey', 'myValue');
echo $session->get('someKey'); // "myValue"
```

<h4 id="getting-all-data">Getting All Data</h4>
```php
$session->set('foo', 'bar');
$session->set('baz', 'blah');
$data = $session->getAll();
echo $data[0]; // "bar"
echo $data[1]; // "blah"
```

<h4 id="checking-if-session-has-key">Checking if a Session Has a Key</h4>
```php
echo $session->has('foo'); // 0
$session->set('foo', 'bar');
echo $session->has('foo'); // 1
```

<h4 id="deleting-data">Deleting Data</h4>
```php
$session->delete('someKey');
```

<h4 id="flushing-all-data">Flushing All Data</h4>
```php
$session->flush();
```

<h4 id="flashing-data">Flashing Data</h4>
Let's say you're writing a form that can display any validation errors after submitting, and you'd like to remember these error messages only for the next request.  Use `flash()`:

```php
$session->flash('formErrors', ['Username is required', 'Invalid email address']);
// ...redirect back to the form...
foreach ($session->get('formErrors') as $error) {
    echo htmlentities($error);
}
```

On the next request, the data in "formErrors" will be deleted.  Want to extend the lifetime of the flash data by one more request?  Use `reflash()`.

<h4 id="regenerating-the-id">Regenerating the Id</h4>
```php
$session->regenerateId();
```

<h2 id="session-handlers">Session Handlers</h2>
**Session handlers** are what actually read and write session data from some form of storage, eg text files, cache, or cookies.  All Opulence handlers implement `\SessionHandlerInterface` (built-into PHP).  Typically, they read and write session data using [middleware](#middleware).  The following are session handlers built-into Opulence:

* `Opulence\Sessions\Handlers\FileSessionHandler`
  * Stores session data to plain-text files
* `Opulence\Sessions\Handlers\CacheSessionHandler`
  * Stores session data to some form of [cache](cache), eg Memcached or Redis

<h2 id="middleware">Middleware</h2>
The best place to read and write session data with the `handler` is in middleware.  Opulence comes with a class middleware baked-in:  `Opulence\Framework\Sessions\Http\Middleware\Session`.

> **Note:** This middleware is an abstract class.  If you're using the <a href="https://github.com/opulencephp/Project" target="_blank">skeleton project</a>, you can simply use the middleware `Project\Http\Middleware\Session` to finish extending it.  Otherwise, you can roll your own session middleware.

Typically in middleware, your `handler` will read from session storage using data passed in through the request.  After the request has been handled and a response generated, the session data is written back to storage via the `handler`.

<h2 id="using-sessions-in-controllers">Using Sessions in Controllers</h2>
To use sessions in your controllers, simply inject it into the controller's constructor along with a type hint:

```php
namespace MyApp\Http\Controllers;

use Opulence\Sessions\ISession;

class MyController
{
    private $session;

    // The session will be automatically injected into the controller by the router
    public function __construct(ISession $session)
    {
        $this->session = $session;
    }
}
```

<h2 id="id-generators">Id Generators</h2>
If your session has just started or if its data has been invalidated, a new session Id will need to be generated.  These Ids must be cryptographically secure to prevent session hijacking.  If you're using `Opulence\Sessions\Session`, you can either pass in your own Id generator (must implement `Opulence\Sessions\Ids\Generators\IIdGenerator`) or use the default `Opulence\Sessions\Ids\Generators\IdGenerator`.

> **Note:** It's recommended you use Opulence's `IdGenerator` unless you know what you're doing.

<h2 id="encrypting-session-data">Encrypting Session Data</h2>
You might find yourself storing sensitive data in sessions, in which case you'll want to encrypt it.  To do this, use the `useEncryption()` and `setEncrypter()` methods.  `setEncrypter()` requires an instance of `Opulence\Sessions\Handlers\ISessionEncrypter`.  For convenience, if you're also using Opulence's [cryptography library](cryptography#encryption), you can use the `Opulence\Sessions\Handlers\SessionEncrypter` class.

```php
use Opulence\Cryptography\Encryption\Encrypter;
use Opulence\Sessions\Handlers\FileSessionHandler;
use Opulence\Sessions\Handlers\SessionEncrypter;

$sessionEncrypter = new SessionEncrypter(new Encrypter('mySecretApplicationKey'));
$handler = new FileSessionHandler('path/to/my/session/files');
$handler->useEncryption(true);
$handler->setEncrypter($sessionEncrypter);
```

Now, all your session data will be encrypted before being written and decrypted after being read.  [Learn more](cryptography#encryption) about encryption.

<h2 id="configuration">Configuration</h2>
If you're using the <a href="https://github.com/opulencephp/Project" target="_blank">skeleton project</a> and you'd like to configure your session, you can do so in *config/http/sessions.php*.  Your environment config in *config/environment/.env.app.php* contains a variable `SESSION_HANDLER`, which should point to the handler class to use (defaults to `FileSessionHandler`).  For cache-backed sessions, *.env.app.php* also contains the variable `SESSION_CACHE_BRIDGE`.  This should point to the class that implements `Opulence\Cache\ICacheBridge`.
