# Console

## Table of Contents
1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Running Commands](#running-commands)
4. [Getting Help](#getting-help)
5. [Built-In Commands](#built-in-commands)
  1. [app:env](#appenv)
  2. [app:rename](#apprename)
  3. [views:flush](#viewsflush)
6. [Arguments](#arguments)
7. [Options](#options)
  1. [Short Names](#short-names)
  2. [Long Names](#long-names)
  2. [Array Options](#array-options)
8. [Responses](#responses)
9. [Formatters](#formatters)
  1. [Padding](#padding)
10. [Style Elements](#style-elements)
  1. [Built-In Elements](#built-in-elements)
  2. [Custom Elements](#custom-elements)
11. [Prompts](#prompts)
  1. [Confirmation](#confirmation)
  2. [Multiple Choice](#multiple-choice)
12. [Creating Commands](#creating-commands)
13. [Calling From Code](#calling-from-code)
  

## Introduction
Console applications are great for administrative tasks and code generation.  RDev supports a robust console kernel with features like easy-to-create commands, question prompts, and HTML-like syntax for output styling.

## Installation
The easiest way to setup a console application is to clone the [empty Project](https://github.com/ramblingsofadev/Project) repository or use Composer's `composer create-project rdev/Project MY_DIRECTORY`.

## Running Commands
To run commands, type `php rdev COMMAND_NAME` from the directory that RDev is installed in.

## Getting Help
To get help with any command, use the help command.  There are a few different ways to call it:

```
php rdev help COMMAND_NAME
php rdev COMMAND_NAME -h
php rdev COMMAND_NAME --help
```

## Built-In Commands
A good place to start is to run `php rdev` in the directory you installed RDev.  This will list the commands registered to the console.  Out of the box, a few commands come bundled with RDev:

#### app:env
Displays the current application environment name, eg "Production" or "Development".

#### app:rename
When you install RDev, the default namespace is "Project".  Use this command to change this to something more fitting to your particular project.  This will update namespaces, bootstrapper names, the directory under "app", and the composer.json PSR-4 settings.

#### views:flush
If you also use RDev's HTTP kernel and view template, you can use this command to clear the view cache.  This is handy for when you've made updates to your views.
  
## Arguments
Console commands can accept arguments from the user.  Arguments can be required, optional, and/or arrays.  You specify the type by bitwise OR-ing the different arguments types.  Array arguments allow a variable number of arguments to be passed in, like "php rdev foo arg1 arg2 arg3 ...".  The only catch is that array arguments must be the last argument defined for the command.

Let's take a look at an example argument:

```php
use RDev\Console\Requests;

// The argument will by required and an array
$type = Requests\ArgumentTypes::REQUIRED | Requests\ArgumentTypes::IS_ARRAY;
// The description argument is used by the help command
$argument = new Argument("foo", $type, "The foo argument");
```

>**Note:** Like array arguments, optional arguments must appear after any required arguments. 

## Options
You might want different behavior in your command depending on whether or not an option is set.  This is possible using `RDev\Console\Requests\Option`.  Options have two formats:

1. Short, eg "-h"
2. Long, eg "--help"

#### Short Names
Short option names are always a single letter.  Multiple short options can be grouped together.  For example, `-rf` means that options with short codes "r" and "f" have been specified.  The default value will be used for short options.

#### Long Names
Long option names can specify values in two ways:  `--foo=bar` or `--foo bar`.  If you only specify `--foo` for an optional-value option, then the default value will be used.

#### Array Options
Options can be arrays, eg `--foo=bar --foo=baz` will set the "foo" option to `["bar", "baz"]`.

Like arguments, option types can be specified by bitwise OR-ing types together.  Let's look at an example:

```php
use RDev\Console\Requests;

$type = Requests\OptionTypes::REQUIRED_VALUE;
$option = new Requests\Option("foo", "f", Requests\OptionTypes::REQUIRED_VALUE, "The foo option");
```

## Responses
Responses are classes that allow you to write output to an end user.  The different responses classes include:

1. `RDev\Console\Responses\Console`
  * Used to write output to the console
  * The response used by default
2. `RDev\Console\Responses\Silent`
  * Used when we don't want any output to be written
  * Useful for when one command calls another
  
Each response offers three methods:

1. `write()`
  * Writes a message to the existing line
2. `writeln()`
  * Writes a message to a new line
3. `clear()`
  * Clears the current screen
  * Only works in `Console` response

## Formatters
Formatters are great for nicely-formatting output to the console.

#### Padding
The `RDev\Console\Responses\Formatters\Padding` formatter allows you to create column-like output.  It accepts an array of column values.  The second parameter is a callback that will format each row's contents.  Let's look at an example:
 
```php
use RDev\Console\Responses\Formatters;

$paddingFormatter = new Formatters\Padding();
$rows = [
    ["George", "Carlin"],
    ["Chris", "Rock"],
    ["Jim", "Gaffigan"]
];
$paddingFormatter->format($rows, function($row)
{
    return $row[0] . " - " . $row[1];
});
```

This will return:
```
George - Carlin
Chris  - Rock
Jim    - Gaffigan
```

The `format()` method accepts options to add the padding before each string, change the padding character, and change the end-of-line character.

## Prompts
Prompts are great for asking users for input beyond what is accepted by arguments.  For example, you might want to confirm with a user before doing an administrative task, or you might ask her to select from a list of possible choices.  Prompts accept `RDev\Console\Prompts\Question\IQuestion` objects.

#### Confirmation
To ask a user to confirm an action with a simple "y" or "yes", use an `RDev\Console\Prompts\Questions\Confirmation`:

```php
use RDev\Console\Prompts;
use RDev\Console\Prompts\Questions;
use RDev\Console\Responses\Formatters;

$prompt = new Prompts\Prompt(new Formatters\Padding());
// This will return true if the answer began with "y" or "Y"
$prompt->ask(new Questions\Confirmation("Are you sure you want to continue?"));
```

#### Multiple Choice
Multiple choice questions are great for listing choices that might otherwise be difficult for a user to remember.  An `RDev\Console\Prompts\Questions\MultipleChoice` accepts question text and a list of choices:

```php
use RDev\Console\Prompts\Questions;

$choices = ["Boeing 747", "Boeing 757", "Boeing 787"];
$question = new Questions\MultipleChoice("Select your favorite airplane", $choices);
$prompt->ask($question);
```

This will display:

```php
Select your favorite airplane
  1) Boeing 747
  2) Boeing 757
  3) Boeing 787
  > 
```

If the `$choices` array is associative, then the keys will map to values rather than 1)...N).  You can enable multiple answers, too:

```php
$question->setAllowMultipleChoices(true);
```

This allows a user to separate multiple choices with a comma, eg "1,3".

## Style Elements
RDev supports HTML-like style elements to perform basic output formatting like background color, foreground color, boldening, and underlining.  It does this by parsing the string into an *Abstract Syntax Tree*, and then converting each node in the tree into the appropriate ANSI codes.  For example, writing:

```
<b>Hello!</b>
```

...will output "<b>Hello!</b>".  You can even nest elements:

```
<u>Hello, <b>Dave</b></u>
```

..., which will output an underlined string where "Dave" is both bold AND underlined.

#### Built-In Elements
The following elements come built-into RDev:
* &lt;success&gt;&lt;/success&gt;
* &lt;info&gt;&lt;/info&gt;
* &lt;question&gt;&lt;/question&gt;
* &lt;comment&gt;&lt;/comment&gt;
* &lt;error&gt;&lt;/error&gt;
* &lt;fatal&gt;&lt;/fatal&gt;
* &lt;success&gt;&lt;/success&gt;
* &lt;b&gt;&lt;/b&gt;
* &lt;u&gt;&lt;/u&gt;

#### Custom Elements
You can create your own style elements.  Elements are registered to `RDev\Console\Responses\Compilers\Compiler`.  To register a custom element, use a bootstrapper:

```php
namespace MyApp\Bootstrappers\Console;
use RDev\Applications\Bootstrappers;
use RDev\Console\Responses\Compilers;
use RDev\Console\Responses\Formatters\Elements;

class CustomElements extends Bootstrappers\Bootstrappers
{
    public function run(Compilers\Compiler $compiler)
    {
        $compiler->getElements()->add(
            new Elements\Element("foo", new Elements\Style(
                  Elements\Colors::BLACK, Elements\Colors::YELLOW, [Elements\TextStyles::BOLD]
            ))
        );
    }
}
```

## Creating Commands
You can create your own commands and register them to the application.  You do this by extending `RDev\Console\Commands\Command`.  To get going, you only need to define two methods:

1. `define()`
  * Used to set the name, description, arguments, and options
2. `doExecute()`
  * Actually executes the command and writes any output to the input `Response` object
  
Let's define a simple command that greets a person and optionally shouts the greeting:

```php
namespace MyApp\Console\Commands;
use RDev\Console\Commands;
use RDev\Console\Requests;
use RDev\Console\Responses;

class Greeting extends Commands\Command
{
    protected function define()
    {
        $this->setName("greet")
            ->setDescription("Greets a person")
            ->addArgument(new Requests\Argument(
                "name", Requests\ArgumentTypes::REQUIRED, "The name to greet"
            ))
            ->addOption(new Requests\Option(
                "yell", "y", Requests\OptionTypes::OPTIONAL_VALUE, "Whether or not to yell the greeting", "yes"
            ));
    }
    
    protected function doExecute(Responses\IResponse $response)
    {
        $greeting = "Hello, " . $this->getArgumentValue("name");
        
        if($this->getOptionValue("yell") == "yes")
        {
            $greeting = strtoupper($greeting);
        }
        
        $response->writeln($greeting);
    }
}
```

To call this command, run:

```
php rdev greet Dave -y
```

This will output:

```
HELLO, DAVE
```

> **Note:** You MUST at least define a name for each command.

To register this command with our application, simply add its fully-qualified name to the array in "configs/console/commands.php".

## Calling From Code
It's possible to call a command from another command:

```php
namespace MyApp\Console\Commands;
use RDev\Console\Commands;
use RDev\Console\Responses;

class MyCommand extends Commands\Command
{
    protected function define()
    {
        // Define the command...
    }
    
    protected function doExecute(Responses\IResponse $response)
    {
        // Call another command
        $this->commands->call("foo", $response, ["argument1Value"], ["--option1Value"]);
    }
}
```

You must list the arguments in the same order they were defined in the command.  If you want to call the other command but not write its output, use the `RDev\Console\Responses\Silent` response.

> **Note:** If a command is being called by a lot of other commands, it might be best to refactor its actions into a separate class.  This way, it can be used by multiple commands without the extra overhead of calling console commands through PHP code.