# Validation

## Table of Contents
1. [Introduction](#introduction)
2. [Rules](#rules)
  1. [Conditional Rules](#conditional-rules)
  2. [Creating Custom Rules](#creating-custom-rules)
      1. [Using Objects](#using-objects)
      2. [Using Callables](#using-callables)
  3. [Halting Validation](#halting-validation)
3. [Error messages](#error-messages)
  1. [Error Message Placeholders](#error-message-placeholders)
4. [Validating Form Input](#validating-form-input)
5. [Validating Models](#validating-models)
6. [Skeleton Project Examples](#skeleton-project-examples)
  1. [Error Message Configuration](#error-message-configuration)
  2. [Controller Example](#validation-in-controller)
  3. [Console Command Example](#validation-in-console-command)
7. [Built-In Rules](#built-in-rules)

<h2 id="introduction">Introduction</h2>
Validating data is a fundamental part of every web application.  Whether it be form data or a single value, Opulence makes it easy to validate your data using a fluent syntax.  For example, want to verify a password is set and matches the confirmation password?  Easy:

```php
use Opulence\Validation\Rules\Errors\Compilers\Compiler;
use Opulence\Validation\Rules\Errors\ErrorTemplateRegistry;
use Opulence\Validation\Rules\Factories\RulesFactory;
use Opulence\Validation\Rules\RuleExtensionRegistry;
use Opulence\Validation\Validator;

// Set some error message templates
$errorTemplateRegistry = new ErrorTemplateRegistry();
$errorTemplateRegistry->registerErrorTemplatesFromConfig([
    'required'    => 'The :field input is required',
    'equalsField' => 'The :field input must match the :other input'
]);

// Create the components
$rulesFactory = new RulesFactory(
    new RuleExtensionRegistry(),
    $errorTemplateRegistry,
    new Compiler()
);
$validator = new Validator($rulesFactory);

// Set some rules for the "password" field
$validator->field('password')
    ->required()
    ->equalsField('confirm-password');

// Validate the input
if (!$validator->isValid(['password' => '1337', 'confirm-password' => 'asdf'])) {
    print_r($validator->getErrors()->getAll());
}
```

> **Note:** To make it easier to create validators, if you're using the [skeleton project](#skeleton-project-examples), an `Opulence\Validation\Factories\IValidatorFactory` is created and bound to the IoC container for you.

Opulence's validation library is framework-agnostic, making it easy to use with both Opulence and other frameworks.

<h2 id="rules">Rules</h2>
Whenever you call `Opulence\Validation\Validator::field()`, a `Rules` object will be created.  It contains a bunch of [built-in rules](#built-in-rules) as well as methods to get any errors for the field.  Most methods are chainable, letting you build up the rules like this:

```php
$validator->field('to')
    ->required()
    ->email();
```

<h4 id="conditional-rules">Conditional Rules</h4>
Sometimes, you may only want to apply a rule if certain conditions are met.  To specify a condition, call `condition()` on the `Rules` object.  It accepts a `callable` with two parameters:

1. The value of the field
2. A list of all fields being validated

The `callable` should return `true` if the condition has been met, otherwise `false`.  Any rule added after `condition()` will only be run if the condition is met.  If you'd like to end the list of conditional rules and add a non-conditional rule, call `endCondition()` before adding the non-conditional rule.

<h5 id="conditional-rules-example">Conditional Rule Example</h5>
Let's say there are two inputs:  a dropdown specifying what type of contact information we're providing, and a text box with the actual contact information.  If the contact type is "email", we want to force the contact information to be a valid email:

```php
$validator->field('contact-info')
    ->condition(function ($value, array $inputs) {
        return $inputs['contact-type'] == 'email';
    })
    ->email();
$validator->isValid([
    'contact-info' => 'foo@bar.com',
    'contact-type' => 'email'
]);
```

Since "contact-type" was "email", the condition was met, and the "email" rule was run.

<h4 id="creating-custom-rules">Creating Custom Rules</h4>
Each rule in `Rules` implements `Opulence\Validation\Rules\IRule`, which provides two methods:

* `getSlug()`
  * Gets the short name for the rule, eg `equalsField` (must only contain alphanumeric characters and underscores)
  * Calling `Rules::{slug}()` will add your rule to the field
  * Used to bind error messages to rules
* `passes($value, array $allValues)`
  * Returns `true` if the rule passes, otherwise `false`

If you'd like to add a custom rule, you can use `RuleExtensionRegistry::registerRule()`.  It accepts either:

* A rule object implementing `IRule`
* A `callable` with parameters for the field value and an array of all field values

<h5 id="using-objects">Using Objects</h5>
If your rule needs to accept extra arguments, such as a value to compare to, implement `IRuleWithArgs` instead of `IRule`.  Let's look at an example of a rule that forces an email to be from a certain domain:

```php
use Opulence\Validation\Rules\IRuleWithArgs;

class EmailDomainRule implements IRuleWithArgs
{
    protected $domain = null;

    public function getErrorPlaceholders() : array
    {
        return ['domain' => $this->domain];
    }

    public function getSlug() : string
    {
        return 'emailDomain';
    }

    public function passes($value, array $allValues = []) : bool
    {
        if ($this->domain === null) {
            throw new LogicException('Email domain not set');
        }

        // Check if the value is even a valid email address
        if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
            return false;
        }

        return preg_match(
            '/@'{preg_quote($this->domain, '/')}$/",
            $value
        ) === 1;
    }

    public function setArgs(array $args)
    {
        if (count($args) == 0 || !is_string($args[0])) {
            throw new InvalidArgumentException("Must pass a valid domain");
        }

        $this->domain = $args[0];
    }
}
```

You can now register this rule.:

```php
$ruleExtensionRegistry->registerRule(new EmailDomainRule);
```

If you're using the <a href="https://github.com/opulencephp/Project" target="_blank">skeleton project</a>, do this registration in the `Project\Application\Bootstrappers\Validation\ValidatorBootstrapper::registerRuleExtensions()` method.  You could also add a custom error message to *resources/lang/en/validation.php*.  The rule we created defined an error placeholder "domain", which we can use in our error message:

```php
return [
    // ...Other error message templates
    'domain' => 'The :field did not belong to the :domain domain'
];
```

To use the extension, simply call `$validator->field("FIELD_NAME")->{slug}()`:

```php
$validator->field('some-email-address')
    ->emailDomain('gmail.com');
```

<h5 id="using-callables">Using Callables</h5>
When registering a `callable`, you must give it a slug:

```php
$rule = function ($value, array $allValues = []) {
    return $value == 'Dave';
};
$ruleExtensionRegistry->registerRule($rule, 'coolName');
$validator->field('name')
    ->coolName();
```

<h4 id="halting-validation">Halting Validation</h4>
Sometimes, you may want to stop validating a field after its first rule failure.  Simply pass `true` to the second parameter in `Validator::isValid()`:

```php
$validator->field('email')
    ->required()
    ->email();
$validator->isValid(['email' => null], true);
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
$validator->getErrors()->get('FIELD_NAME');
```

Error message templates are bound to a slug in the `Opulence\Validation\Rules\Errors\ErrorTemplateRegistry`:

```php
use Opulence\Validation\Rules\Errors\ErrorTemplateRegistry;

$errorTemplateRegistry = new ErrorTemplateRegistry();
$errorTemplateRegistry->registerErrorTemplatesFromConfig([
    'required' => 'The :field input is required',
    'email.required' => 'We need your email address'
]);
```

All "required" rules that fail will now have the first error message.  Specifying `email.required` will override the global error message for "required" rules, but only for the "email" field.  `:field` will automatically be populated with the name of the field that failed.

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

    public function getErrorPlaceholders() : array
    {
        $dayName = DateTime::createFromFormat('!N', $this->comparisonDay)->format('l');

        return ['day' => $dayName];
    }

    public function getSlug() : string
    {
        return 'day';
    }

    public function passes($value, array $allValues = []) : bool
    {
        return (new DateTime($value))->format('N') == $this->comparisonDay;
    }

    public function setArgs(array $args)
    {
        $this->comparisonDay = $args[0];
    }
}
```

We can then bind an error message to the rule:

```php
$errorTemplateRegistry->registerGlobalErrorTemplate('day', 'Selected day must be a :day');
```

Now, whenever our rule fails, the nicely-formatted day name will appear in the error message, eg "Selected day must be a Monday".

<h2 id="validating-form-input">Validating Form Input</h2>
First, set up your rules for your fields:

```php
$validator->field('first-name')
    ->required();
$validator->field('last-name')
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

<h2 id="validating-models">Validating Models</h2>
You can validate your models with Opulence's validation library.  One easy way is to extend `Opulence\Validation\Models\ModelState`.  It contains two abstract methods:

* `getModelProperties($model)`
  * Returns mapping of property names => property values in the model
* `registerFields(IValidator $validator)`
  * Registers the fields' rules to the validator

Let's take a look at an example user model state:

```php
namespace MyApp\Validation\Models;

use Opulence\Validation\Models\ModelState;

class UserModelState extends ModelState
{
    protected function getModelProperties($model) : array
    {
        return [
            'id' => $model->getId(),
            'name' => $model->getName(),
            'email' => $model->getEmail()
        ];
    }

    protected function registerFields(IValidator $validator)
    {
        $validator->field('id')
            ->integer();
        $validator->field('name')
            ->required();
        $validator->field('email')
            ->email();
    }
}
```

Now, let's check if a user model is valid:

```php
use MyApp\User;

// We will assume that $validatorFactory was already instantiated
$user = new User(123, 'Dave', 'foo@bar.com');
$modelState = new UserModelState($user, $validatorFactory);

if (!$modelState->isValid()) {
    print_r($modelState->getErrors()->getAll());
}
```

> **Note:** `ModelState::getErrors()` returns an instance of `ErrorCollection`.

<h2 id="skeleton-project-examples">Skeleton Project Examples</h2>

<h4 id="error-message-configuration">Error Message Configuration</h4>
If you're using the <a href="https://github.com/opulencephp/Project" target="_blank">skeleton project</a>, you will find some default error message templates in *config/resources/lang/en/validation.php*.  You are free to edit them as you'd like.

The skeleton project comes with `Project\Bootstrappers\Validation\ValidatorBootstrapper`, which sets your default error message templates and registers your custom rules.  It also binds the validator factory to the IoC container.  If you'd like to use validators in your controllers or console commands, simply inject `IValidatorFactory` via the controller and command constructors, respectively:

<h4 id="validation-in-controller">Controller Example</h4>
```php
use Opulence\Validation\Factories\IValidatorFactory;

class MyController
{
    private $validatorFactory = null;

    public function __construct(IValidatorFactory $validatorFactory)
    {
        $this->validatorFactory = $validatorFactory;
    }

    public function login()
    {
        $validator = $this->validatorFactory->createValidator();
        // You can now use $validator to validate input
    }
}
```

<h4 id="validation-in-console-command">Console Command Example</h4>
```php
use Opulence\Console\Commands\Command;
use Opulence\Console\Responses\IResponse;
use Opulence\Validation\Factories\IValidatorFactory;

class MyCommand extends command
{
    private $validatorFactory = null;

    public function __construct(IValidatorFactory $validatorFactory)
    {
        parent::__construct();

        $this->validatorFactory = $validatorFactory;
    }

    protected function define()
    {
        $this->setName('my:command')
            ->setDescription('My command that uses validation');
    }

    protected function doExecute(IResponse $response)
    {
        $validator = $this->validatorFactory->createValidator();
        // You can now use $validator to validate input
    }
}
```

<h2 id="built-in-rules">Built-In Rules</h2>

The following rules are built into Opulence:

* `alpha()`
  * Checks if the value only contains alphabet characters
* `alphaNumeric()`
  * Checks if the value only contains alpha-numeric characters
* `between($min, $max, $isInclusive = true)`
  * Checks if the value is between two extremes
  * If the third parameter is true, then the extremes are considered inclusive
* `callback(callable $callback)`
  * Checks if the callback returns true
  * The callback must accept the value and an array of all values as parameters
* `date($format)`
  * Checks if the value is a date/time string in the input format
  * If `$format` is an array, then this will return true if at least one format is matched
* `email()`
  * Checks if the value is an email
* `equals($expected)`
  * Checks if the value matches the expected value
* `equalsField($otherField)`
  * Checks if the value matches another field's value
* `in(array $array)`
  * Checks if the value is in the array
* `integer()`
  * Checks if the value is an integer
* `ipAddress()`
  * Checks if the value is an IP address
* `max($max, $isInclusive = true)`
  * Checks if the value is less than the input value
  * If the second parameter is true, then the maximum is considered inclusive
* `min($min, $isInclusive = true)`
  * Checks if the value is greater than the input value
  * If the second parameter is true, then the minimum is considered inclusive
* `notIn(array $array)`
  * Checks if the value is not in the array
* `numeric()`
  * Checks if the value is numeric
* `regex($regex)`
  * Checks if the value matches a regular expression
  * Your regex must include delimiters, eg `/[a-z]/`
* `required()`
  * Checks if the value is not empty, null, an empty array, or an empty `Countable`
