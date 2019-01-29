# Creating Console Commands

## Table of Contents
1. [Introduction](#introduction)
2. [Arguments](#arguments)
3. [Options](#options)
  1. [Short Names](#short-names)
  2. [Long Names](#long-names)
  2. [Array Options](#array-options)
4. [Creating Commands](#creating-commands)
  1. [Dependencies](#dependencies)
  2. [Example](#example)
  3. [Registering Your Command](#registering-your-command)
5. [Calling From Code](#calling-from-code)

<h2 id="introduction">Introduction</h2>

Apex makes it easy to create powerful, custom commands.  From compiling Markdown files to resynchronzing cache with the database, commands simplify administrative components in your application.

<h2 id="arguments">Arguments</h2>

Console commands can accept arguments from the user.  Arguments can be required, optional, and/or arrays.  You specify the type by bitwise OR-ing the different arguments types.  Array arguments allow a variable number of arguments to be passed in, like "php apex foo arg1 arg2 arg3 ...".  The only catch is that array arguments must be the last argument defined for the command.

Let's take a look at an example argument:

```php
use Opulence\Console\Requests\Argument;
use Opulence\Console\Requests\ArgumentTypes;

// The argument will be required and an array
$type = ArgumentTypes::REQUIRED | ArgumentTypes::IS_ARRAY;
// The description argument is used by the help command
$argument = new Argument('foo', $type, 'The foo argument');
```

>**Note:** Like array arguments, optional arguments must appear after any required arguments.

<h2 id="options">Options</h2>

You might want different behavior in your command depending on whether or not an option is set.  This is possible using `Opulence\Console\Requests\Option`.  Options have two formats:

1. Short, eg "-h"
2. Long, eg "--help"

<h4 id="short-names">Short Names</h4>

Short option names are always a single letter.  Multiple short options can be grouped together.  For example, `-rf` means that options with short codes "r" and "f" have been specified.  The default value will be used for short options.

<h4 id="long-names">Long Names</h4>

Long option names can specify values in two ways:  `--foo=bar` or `--foo bar`.  If you only specify `--foo` for an optional-value option, then the default value will be used.

<h4 id="array-options">Array Options</h4>

Options can be arrays, eg `--foo=bar --foo=baz` will set the "foo" option to `["bar", "baz"]`.

Like arguments, option types can be specified by bitwise OR-ing types together.  Let's look at an example:

```php
use Opulence\Console\Requests\Option;
use Opulence\Console\Requests\OptionTypes;

$type = OptionTypes::IS_ARRAY | OptionTypes::REQUIRED_VALUE;
$option = new Option('foo', 'f', $types, 'The foo option');
```

<h2 id="creating-commands">Creating Commands</h2>

You can create your own commands and register them to the application.  You do this by extending `Opulence\Console\Commands\Command`.  To get going, you only need to define two methods:

1. `define()`
  * Used to set the name, description, arguments, and options
2. `doExecute()`
  * Actually executes the command and writes any output to the input `Response` object

<h4 id="dependencies">Dependencies</h4>

If your command requires some dependencies in its constructor, the IoC container will resolve them and inject them into the constructor.

> **Note:** If you define your own constructor in your command class, you MUST call `parent::construct()` from within it.

<h4 id="example">Example</h4>

Let's define a simple command that greets a person and optionally shouts the greeting:

```php
namespace Project\Application\Console\Commands;

use Opulence\Console\Commands\Command;
use Opulence\Console\Requests\Argument;
use Opulence\Console\Requests\ArgumentTypes;
use Opulence\Console\Requests\Option;
use Opulence\Console\Requests\OptionTypes;
use Opulence\Console\Responses\IResponse;

class Greeting extends Command
{
    protected function define()
    {
        $this->setName('greet')
            ->setDescription('Greets a person')
            ->addArgument(new Argument(
                'name', ArgumentTypes::REQUIRED, 'The name to greet'
            ))
            ->addOption(new Option(
                'yell', 'y', OptionTypes::OPTIONAL_VALUE, 'Yell the greeting?', 'yes'
            ));
    }

    protected function doExecute(IResponse $response)
    {
        $greeting = 'Hello, ' . $this->getArgumentValue('name');

        if ($this->getOptionValue('yell') == 'yes') {
            $greeting = strtoupper($greeting);
        }

        $response->writeln($greeting);
    }
}
```

To call this command, run:

```
php apex greet Dave -y
```

This will output:

```
HELLO, DAVE
```

> **Note:** You MUST at least define a name for each command.

<h4 id="registering-your-command">Registering Your Command</h4>

To register this command with our application, simply add its fully-qualified name to the array in *config/console/commands.php*.

<h2 id="calling-from-code">Calling From Code</h2>

It's possible to call a command from another command:

```php
namespace Project\Application\Console\Commands;

use Opulence\Console\Commands\Command;
use Opulence\Console\Responses\IResponse;

class MyCommand extends Command
{
    protected function define()
    {
        // Define the command...
    }

    protected function doExecute(IResponse $response)
    {
        // Call another command
        $this->commandCollection->call('foo', $response, ['argument1Value'], ['--option1Value']);
    }
}
```

You must list the arguments in the same order they were defined in the command.  If you want to call the other command but not write its output, use the `Opulence\Console\Responses\SilentResponse` response.

> **Note:** If a command is being called by a lot of other commands, it might be best to refactor its actions into a separate class.  This way, it can be used by multiple commands without the extra overhead of calling console commands through PHP code.
