# Testing Console Applications

## Table of Contents
1. [Introduction](#introduction)
2. [call()](#call)
3. [Response Assertions](#response-assertions)
  1. [isError()](#assert-status-code-is-error)
  2. [isFatal()](#assert-status-code-is-fatal)
  3. [isOK()](#assert-status-code-is-ok)
  4. [isWarning()](#assert-status-code-is-warning)
  5. [outputEquals()](#assert-output-equals)
  6. [statusCodeEquals()](#assert-status-code-equals)

<h2 id="introduction">Introduction</h2>
Opulence comes with the ability to integration test your console commands using `Opulence\Framework\Testing\PhpUnit\Console\ApplicationTestCase`.  With it, you can test the output of your commands, simulate responses to prompt questions, and check the status code of the kernel.  Simply extend `ApplicationTestCase` in your test classes, and you'll inherit the following methods to help test your console application:

> **Note:** If you need to define a `setUp()` or `tearDown()` method in your test, make sure to call `parent::setUp()` or `parent::tearDown()`.

<h2 id="call">call()</h2>
Allows you to simulate a call to a console command.  It accepts:

* The command name
* The array of argument values
* The array of option values
* The answer or array of answers to use in any prompts
* Whether or not to style the output

Let's say that our command has a `Confirmation` question that we want to try testing with `true` and `false` responses:

```php
public function testConfirmation()
{
    $this->call("app:rename", ["Project", "MyApp"], [], true);
    $this->assertOutputEquals("Updated name successfully");
    $this->call("app:rename", ["Project", "MyApp"], [], false);
    $this->assertOutputEquals("Aborted");
}
```

> **Note:**  This must be called before any assertions in this class.

<h2 id="response-assertions">Response Assertions</h2>
You can run various assertions on the response returned by the `Kernel`.  To do so, simply use `$this->assertResponse`.

<h4 id="assert-status-code-is-error">isError()</h4>
Asserts that the status code of the last command is an error:

```php
public function testStatusCode()
{
    $this->call("errorcommand")
        ->assertResponse
        ->isError();
}
```

<h4 id="assert-status-code-is-fatal">iIsFatal()</h4>
Asserts that the status code of the last command is fatal:

```php
public function testStatusCode()
{
    $this->call("badcommand")
        ->assertResponse
        ->isFatal();
}
```

<h4 id="assert-status-code-is-ok">isOK()</h4>
Asserts that the status code of the last command is OK:

```php
public function testStatusCode()
{
    $this->call("goodcommand")
        ->assertResponse
        ->isOK();
}
```

<h4 id="assert-status-code-is-warning">isWarning()</h4>
Asserts that the status code of the last command is a warning:

```php
public function testStatusCode()
{
    $this->call("warningcommand")
        ->assertResponse
        ->isWarning();
}
```

<h4 id="assert-output-equals">outputEquals()</h4>
Asserts that the output of the last command equals an expected value:

```php
public function testOutputIsCorrect()
{
    $this->call("hello")
        ->assertResponse
        ->outputEquals("Hello, world");
}
```

<h4 id="assert-status-code-equals">statusCodeEquals()</h4>
Asserts that the status code of the last command equals an expected value:

```php
public function testStatusCode()
{
    $this->call("hello")
        ->assertResponse
        ->statusCodeEquals(StatusCodes::WARNING);
}
```