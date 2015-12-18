# Validation

## Table of Contents
1. [Introduction](#introduction)
2. [Rules](#rules)
  1. [Conditional Rules](#conditional-rules)
  2. [Extending Rules](#extending-rules)
      1. [Using Objects](#using-objects)
      2. [Using Callables](#using-callables)
  3. [Halting Validation](#halting-validation)
3. [Error messages](#error-messages)
  1. [Error Message Placeholders](#error-message-placeholders)
4. [Validating Form Input](#validating-form-input)
5. [Skeleton Project Examples](#skeleton-project-examples)
  1. [Error Message Configuration](#error-message-configuration)
  2. [Controller Example](#validation-in-controller)
  3. [Console Command Example](#validation-in-console-command)
6. [Built-In Rules](#built-in-rules)
  
<h2 id="introduction">Introduction</h2>
Validating data is a fundamental part of every web application.  Whether it be form data or a single value, Opulence makes it easy to validate your data using a fluent syntax.  For example, want to verify a password is set and matches the confirmation password?  Easy:

```php
use Opulence\Validation\Rules\Errors\Compilers\Compiler;
use Opulence\Validation\Rules\Errors\ErrorTemplateRegistry;
use Opulence\Validation\Rules\Factories\RulesFactory;
use Opulence\Validation\Rules\RuleExtensionRegistry;
use Opulence\Validation\Validator;

// Create our components
$ruleExtensionRegistry = new RuleExtensionRegistry();
$errorTemplateRegistry = new ErrorTemplateRegistry();
$errorTemplateCompiler = new Compiler();
$rulesFactory = new RulesFactory(
    $ruleExtensionRegistry,
    $errorTemplateRegistry,
    $errorTemplateCompiler
);
$validator = new Validator($rulesFactory, $ruleExtensionRegistry);

// Specify some error message templates
$errorTemplateRegistry->registerErrorTemplatesFromConfig([
    "required" => "The :field input is required",
    "equals_field" => "The :field input must match the :other input"
]);

// Set up our rules
$validator->field("password")
    ->required()
    ->equalsField("confirm-password");
    
// Do some validation
if (!$validator->isValid(["password" => "1337", "confirm-password" => "asdf"]) {
    echo "<ul>";

    foreach ($validator->getErrors()->getAll() as $field => $fieldErrors) {
        foreach ($fieldErrors as $error) {
            echo "<li>$error</li>";
        }
    }
    
    echo "</ul>";
}
```

> **Note:** If you're using the [skeleton project](#skeleton-project-examples), all of the components are created and bound to the IoC container for you.

Opulence's validation library is framework-agnostic, making it easy to use with both Opulence and other frameworks.

<h2 id="rules">Rules</h2>
Whenever you call `Opulence\Validation\Validator::field()`, a `Rules` object will be created.  It contains a bunch of [built-in rules](#built-in-rules) as well as methods to get any errors for the field.

<h4 id="conditional-rules">Conditional Rules</h4>
Sometimes, you may only want to apply a rule if certain conditions are met.  To specify a condition, call `condition()` on the `Rules` object.  It accepts a `callable` with two parameters:

1. The value of the field
2. A list of all fields being validated.  

The `callable` should return `true` if the condition has been met, otherwise `false`.  Any rule added after `condition()` will only be run if the condition is met.  If you'd like to end the list of conditional rules and add a non-conditional rule, call `endCondition()` before adding the non-conditional rule.

<h5 id="conditional-rules-example">Example</h5>
Let's say there are two inputs:  a dropdown specifying what type of contact information we're providing, and a text box with the actual contact information.  If the contact type is "email", we want to force the contact information to be a valid email:

```php
$validator->field("contact-info")
    ->condition(function ($value, array $inputs) {
        return $inputs["contact-type"] == "email";
    })
    ->email();
$validator->isValid([
    "contact-info" => "foo@bar.com", 
    "contact-type" => "email"
]);
```

Since "contact-type" was "email", the condition was met, and the "email" rule was run.

<h4 id="extending-rules">Extending Rules</h4>
Each rule in `Rules` implements `Opulence\Validation\Rules\IRule`, which provides two methods:

* `getSlug()`
  * Gets the snake-case short name for the rule, eg `equals_field` (must only contain alphanumeric characters and underscores)
  * Used to bind error messages to rules
* `passes($value, array $allValues)`
  * Returns `true` if the rule passes, otherwise `false`
  
If you'd like to add a custom rule, you can use `Validator::registerRule()`.  It accepts either:
 
* A rule object implementing `IRule`
* A `callable` with parameters for the field value and an array of all field values

<h5 id="using-objects">Using Objects</h5>
If your rule needs to accept extra arguments, such as a value to compare to, implement `IRuleWithArgs`:

```php
namespace MyApp\Validation\Rules;

use Opulence\Validation\Rules\IRuleWithArgs;

class NotInArrayRule implements IRuleWithArgs
{
    private $array = [];
    
    public function getSlug()
    {
        return "not_in_array";
    }
    
    public function passes($value, array $allValues = [])
    {
        return !in_array($value, $this->array);
    }

    public function setArgs(array $args)
    {
        $this->array = (array)$args[0];
    }
}
```

You can now register this rule:

```php
$validator->registerRule(new NotInArrayRule);
```

To use the extension, simply call `$validator->field("FIELD_NAME")->{slug}()`:

```php
$validator->field("some-array")
    ->not_in_array(["not", "allowed"]);
```

<h5 id="using-callables">Using Callables</h5>
When registering a `callable`, you must give it a slug:

```php
$rule = function ($value, array $allValues = []) {
    return $value == "Dave";
};
$validator->registerRule($rule, "cool_name");
$validator->field("name")
    ->cool_name();
```

<h4 id="halting-validation">Halting Validation</h4>
Sometimes, you may want to stop validating a field after the first failure.  Simply pass `true` to the second parameter in `Validator::isValid()`:

```php
$validator->field("email")
    ->required()
    ->email();
$validator->isValid(["email" => null], true);
```

In this example, because the "email" field was null, it fails the "required" rule.  This means the "email" rule will never be run.

<h2 id="error-messages">Error Messages</h2>
Calling `Validator::getErrors()` after `Validator::isValid()` will return an `Opulence\Validation\Rules\Errors\ErrorCollection` with any error messages from validation.

To grab all errors, use:
```php
$validator->getErrors()->getAll();
```

To get a specific field's error message, use:
```php
$validator->getErrors()->get("FIELD_NAME");
```

Error message templates are bound to a slug in the `Opulence\Validation\Rules\Errors\ErrorTemplateRegistry`:

```php
use Opulence\Validation\Rules\Errors\ErrorTemplateRegistry;

$errorTemplateRegistry = new ErrorTemplateRegistry();
$errorTemplateRegistry->registerErrorTemplatesFromConfig([
    "required" => "The :field input is required",
    "email.required" => "We need your email address"
]);
```

All "required" rules that fail will now have the first error message.  Specifying `email.required` makes all "required" rules for the "email" field have the second error message.  `:field` will automatically be populated with the name of the field that failed.

<h4 id="error-message-placeholders">Error Message Placeholders</h4>
You can specify placeholders in your error messages using `:NAME_OF_PLACEHOLDER`.  If your rule needs to specify placeholder values, it should also implement `IRuleWithErrorPlaceholders`.  In the `getErrorPlaceholders()` method, you can return a keyed array with the placeholder-name => placeholder-value mappings.
 
Let's take a look at an example of a rule that checks if an input date falls on a particular day (numbered 0-6):

```php
namespace MyApp\Validation\Rules;

use DateTime;
use Opulence\Validation\Rules\IRuleWithArgs;
use Opulence\Validation\Rules\IRuleWithErrorPlaceholders;

class DayRule implements IRuleWithArgs, IRuleWithErrorPlaceholders
{
    private $comparisonDay = null;

    public function getErrorPlaceholders()
    {
        $dayName = DateTime::createFromFormat("!N", $this->comparisonDay)->format("l");
        
        return ["day" => $dayName];
    }
    
    public function getSlug()
    {
        return "day";
    }
    
    public function passes($value, array $allValues = [])
    {
        return (new DateTime($value))->format("N") == $this->comparisonDay;
    }
    
    public function setArgs(array $args)
    {
        $this->comparisonDay = $args[0];
    }
}
```

We can then bind an error message to the rule:

```php
$errorTemplateRegistry->registerGlobalErrorTemplate("day", "Selected day must be a :day");
```

Now, whenever our rule fails, the nicely-formatted day name will appear in the error message, eg "Selected day must be a Monday".

<h2 id="validating-form-input">Validating Form Input</h2>
First, set up your rules for your fields:

```php
$validator->field("first-name")
    ->required();
$validator->field("last-name")
    ->required();
```

Then, if you're using Opulence's [HTTP request wrapper](http-requests-responses), call:

```php
$validator->isValid($request->getPost()->getAll());
```

> **Note:** You can also pass in `$request->getQuery()->getAll()`.

If you aren't using Opulence's HTTP request wrapper, you can pass any of the PHP super globals to the validator:

```php
$validator->isValid($_POST);
```

<h2 id="skeleton-project-examples">Skeleton Project Examples</h2>

<h4 id="error-message-configuration">Error Message Configuration</h4>
If you're using the <a href="https://github.com/opulencephp/Project" target="_blank">skeleton project</a>, you will find some default error message templates in `config/resources/lang/en/validation.php`.  You are free to edit them as you'd like.

The `Project\Bootstrappers\Validation\ValidatorBootstrapper` binds the validator to `Opulence\Validation\IValidator`.  If you'd like to use the validator in your controllers or console commands, simply inject them via the controller and command constructors, respectively:

<h4 id="validation-in-controller">Controller Example</h4>
```php
use Opulence\Validation\IValidator;

class MyController
{
    private $validator = null;
    
    public function __construct(IValidator $validator)
    {
        $this->validator = $validator;
    }
    
    public function login()
    {
        // You can now use $this->validator to validate input
    }
}
```

<h4 id="validation-in-console-command">Console Command Example</h4>
```php
use Opulence\Console\Commands\Command;
use Opulence\Console\Responses\IResponse;
use Opulence\Validation\IValidator;

class MyCommand extends command
{
    private $validator = null;
    
    public function __construct(IValidator $validator)
    {
        parent::__construct();
    
        $this->validator = $validator;
    }
    
    protected function define()
    {
        $this->setName("my:command")
            ->setDescription("My command that uses validation");
    }
    
    protected function doExecute(IResponse $response)
    {
        // You can now use $this->validator to validate input
    }
}
```
 
<h2 id="built-in-rules">Built-In Rules</h2>

> **Note:** We are adding a lot more rules in the coming weeks.

The following rules are built into Opulence:

* `email()`
  * Checks if the value is an email
* `equals($expected)`
  * Checks if the value matches the expected value
* `equalsField($otherField)`
  * Checks if the value matches another field's value
* `required()`
  * Checks if the value is not empty nor null