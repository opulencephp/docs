# Testing Console Applications

## Table of Contents
1. [Introduction](#introduction)
2. [Testing Commands](#testing-commands)
  1. [Prompt Answers](#prompt-answers)
  2. [with() Methods](#with-methods)
3. [Response Assertions](#response-assertions)
  1. [isError()](#assert-status-code-is-error)
  2. [isFatal()](#assert-status-code-is-fatal)
  3. [isOK()](#assert-status-code-is-ok)
  4. [isWarning()](#assert-status-code-is-warning)
  5. [outputEquals()](#assert-output-equals)
  6. [statusCodeEquals()](#assert-status-code-equals)

<h2 id="introduction">Introduction</h2>
Opulence comes with the ability to integration test your console commands using `Opulence\Framework\Testing\PhpUnit\Console\IntegrationTestCase`.  With it, you can test the output of your commands, simulate responses to prompt questions, and check the status code of the kernel.  Simply extend `ApplicationTestCase` in your test classes, and you'll inherit the following methods to help test your console application:

> **Note:** If you need to define a `setUp()` or `tearDown()` method in your test, make sure to call `parent::setUp()` or `parent::tearDown()`.

<h2 id="testing-commands">Testing Commands</h2>
There are two ways to test a command:

1. Call `$this->execute()` and pass in the command name, argument values, option values, prompt answers, and whether or not to style the output
2. Call `$this->command()`, which returns a `CommandBuilder` to build up your command

For example, the following two tests will test identical things:

```php
public function testUsingExecute()
{
    $this->execute("app:rename", ["Project", "MyApp"], [], true)
        ->assertResponse
        ->outputEquals("<success>Updated name successfully</success>");
}

public function testUsingCommand()
{
    $this->command("app:rename")
        ->withArguments(["Project", "MyApp"])
        ->withAnswers(true)
        ->execute()
        ->assertResponse
        ->outputEquals("<success>Updated name successfully</success>");
}
```

<h4 id="prompt-answers">Prompt Answers</h4>  
Let's say that our command has a `Confirmation` question that we want to try testing with `true` and `false` answers.  Simply pass the answers and test the output of each:

```php
public function testConfirmation()
{
    $this->command("app:rename")
        ->withArguments(["Project", "MyApp"])
        ->withAnswers(true)
        ->execute()
        ->assertResponse
        ->outputEquals("<success>Updated name successfully</success>");
    $this->command("app:rename")
        ->withArguments(["Project", "MyApp"])
        ->withAnswers(false)
        ->execute()
        ->assertResponse
        ->outputEquals("");
}
```

<h4 id="with-methods">with() Methods</h4>
The following methods can be used to pass data to your command:

* `withArguments($arguments)`
  * Passes the argument or array of arguments to the command
* `withAnswers($answers)`
  * Passes the prompt answers or array of prompt answers to the command
* `withOptions($options)`
  * Passes the options or array of options to the command
* `withStyle($isStyled)`
  * Sets whether or not the response should be styled

<h2 id="response-assertions">Response Assertions</h2>
You can run various assertions on the response returned by the `Kernel`.  To do so, simply use `$this->assertResponse`.

<h4 id="assert-status-code-is-error">isError()</h4>
Asserts that the status code of the last command is an error:

```php
public function testStatusCode()
{
    $this->execute("errorcommand")
        ->assertResponse
        ->isError();
}
```

<h4 id="assert-status-code-is-fatal">isFatal()</h4>
Asserts that the status code of the last command is fatal:

```php
public function testStatusCode()
{
    $this->execute("badcommand")
        ->assertResponse
        ->isFatal();
}
```

<h4 id="assert-status-code-is-ok">isOK()</h4>
Asserts that the status code of the last command is OK:

```php
public function testStatusCode()
{
    $this->execute("goodcommand")
        ->assertResponse
        ->isOK();
}
```

<h4 id="assert-status-code-is-warning">isWarning()</h4>
Asserts that the status code of the last command is a warning:

```php
public function testStatusCode()
{
    $this->execute("warningcommand")
        ->assertResponse
        ->isWarning();
}
```

<h4 id="assert-output-equals">outputEquals()</h4>
Asserts that the output of the last command equals an expected value:

```php
public function testOutputIsCorrect()
{
    $this->execute("hello")
        ->assertResponse
        ->outputEquals("Hello, world");
}
```

<h4 id="assert-status-code-equals">statusCodeEquals()</h4>
Asserts that the status code of the last command equals an expected value:

```php
public function testStatusCode()
{
    $this->execute("hello")
        ->assertResponse
        ->statusCodeEquals(StatusCodes::WARNING);
}
```