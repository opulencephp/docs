# Testing Console Applications

## Table of Contents
1. [Introduction](#introduction)
2. [call()](#call)
3. [assertOutputEquals()](#assert-output-equals)
4. [assertStatusCodeEquals()](#assert-status-code-equals)
5. [assertStatusCodeIsError()](#assert-status-code-is-error)
6. [assertStatusCodeIsFatal()](#assert-status-code-is-fatal)
7. [assertStatusCodeIsOK()](#assert-status-code-is-ok)
8. [assertStatusCodeIsWarning()](#assert-status-code-is-warning)
9. [getOutput()](#get-output)

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

<h2 id="assert-output-equals">assertOutputEquals()</h2>
Asserts that the output of the last command equals an expected value:

```php
public function testOutputIsCorrect()
{
    $this->call("hello");
    $this->assertOutputEquals("Hello, world");
}
```

<h2 id="assert-status-code-equals">assertStatusCodeEquals()</h2>
Asserts that the status code of the last command equals an expected value:

```php
public function testStatusCode()
{
    $this->call("hello");
    $this->assertStatusCodeEquals(StatusCodes::WARNING);
}
```

<h2 id="assert-status-code-is-error">assertStatusCodeIsError()</h2>
Asserts that the status code of the last command is an error:

```php
public function testStatusCode()
{
    $this->call("errorcommand");
    $this->assertStatusCodeIsError();
}
```

<h2 id="assert-status-code-is-fatal">assertStatusCodeIsFatal()</h2>
Asserts that the status code of the last command is fatal:

```php
public function testStatusCode()
{
    $this->call("badcommand");
    $this->assertStatusCodeIsFatal();
}
```

<h2 id="assert-status-code-is-ok">assertStatusCodeIsOK()</h2>
Asserts that the status code of the last command is OK:

```php
public function testStatusCode()
{
    $this->call("goodcommand");
    $this->assertStatusCodeIsOK();
}
```

<h2 id="assert-status-code-is-warning">assertStatusCodeIsWarning()</h2>
Asserts that the status code of the last command is a warning:

```php
public function testStatusCode()
{
    $this->call("warningcommand");
    $this->assertStatusCodeIsWarning();
}
```

<h2 id="get-output">getOutput()</h2>
Gets the output from the last command:

```php
public function testOutput()
{
    $this->call("hello");
    $this->assertNotEquals("Goodbye, world", $this->getOutput());
}
```