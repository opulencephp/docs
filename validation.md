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
4. [Built-In Rules](#built-in-rules)
  
<h2 id="introduction">Introduction</h2>
Validating data is a fundamental part of every web application.  Whether it be form data or a single value, Opulence makes it easy to validate your data using a fluent syntax.  For example, want to verify that a value is an email?  Easy:

```php
$validator->field("user-email")
    ->email();
echo $validator->isValid(["user-email" => "foo@bar.com"]); // 1
```

Opulence's validation library is not tied to another other library, meaning you are free to use it without inheriting a ton of dependencies.

<h2 id="rules">Rules</h2>
Whenever you call `Opulence\Validation\Validator::field()`, a `Rules` object will be created.  It contains a bunch of [built-in rules](#built-in-rules) as well as methods to get any errors for the field.

<h4 id="conditional-rules">Conditional Rules</h4>
Sometimes, you may only want to apply a rule if certain conditions are met.  To specify a condition, call `condition()` on the `Rules` object.  It accepts a `callable` with two parameters:  the value of the field and a list of all fields being validated.  It should return `true` if the condition has been met, otherwise `false`.  Any rule added to `Rules` after `condition()` will be considered conditional.  If you'd like to end the list of conditional rules and add a non-conditional rule, call `endCondition()` before adding the non-conditional rule.

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

<h4 id="extending-rules">Extending Rules</h4>
Each rule in `Rules` implements `Opulence\Validation\Rules\IRule`, which provides two methods:

* `getSlug()`
  * Gets the snake-case short name for the rule, eg `equals_field`
  * Must only contain alphanumeric characters and underscores
  * Used to associate error message templates with rules
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
Sometimes, you may want to stop validating a field after the first failure.  Simply pass `true` into the second parameter in `Validator::isValid()`:

```php
$validator->field("email")
    ->required()
    ->email();
$validator->isValid(["email" => null], true);
```

In this example, because the "email" field was null, it fails the "required" rule.  This means the "email" rule will never be run.

<h2 id="error-messages">Error Messages</h2>
Calling `Validator::getErrors()` after `Validator::isValid()` will return an `Opulence\Validation\Rules\Errors\ErrorCollection` with any error messages.  To grab all errors, use `$validator->getErrors()->getAll()`.  To get a specific field's error message, use `$validator->getErrors()->get("FIELD_NAME")`.

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
If your rule needs to specify placeholder values in your error messages, it should also implement `IRuleWithErrorPlaceholders`.  In the `getErrorPlaceholders()` method, you can return a keyed array with the placeholder-name => placeholder-value mappings.
 
Let's take a look at an example of a rule that checks if an input date falls on a particular day (numbered 0-6).

```php
namespace MyApp\Validation\Rules;

use DateTime;
use Opulence\Validation\Rules\IRule;
use Opulence\Validation\Rules\IRuleWithErrorPlaceholders;

class DayRule implements IRuleWithArgs, IRuleWithErrorPlaceholders
{
    private $comparisonDay = null;

    public function getErrorPlaceholders()
    {
        $dayName = DateTime::createFromFormat("!N", $this->comparisonDay)
            ->format("l");
        
        return ["day" => $dayName];
    }
    
    public function getSlug()
    {
        return "day";
    }
    
    public function passes($value, array $allValues = [])
    {
        $date = DateTime::createFromFormat("!N", $value);
        
        return $date->format("N") == $this->comparisonDay;
    }
    
    public function setArgs(array $args)
    {
        $this->comparisonDay = $args[0];
    }
}
```

In our `ErrorTemplateRegistry`, we can then bind an error message to the rule:

```php
$errorTemplateRegistry->registerGlobalErrorTemplate("day", "Selected day must be a :day");
```

Now, whenever our rule fails, the nicely-formatted day name will appear in the error message, eg "Selected day must be a Monday".
 
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