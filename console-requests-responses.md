# Console Requests/Responses

## Table of Contents
1. [Prompts](#prompts)
  1. [Confirmation](#confirmation)
  2. [Multiple Choice](#multiple-choice)
2. [Responses](#responses)
3. [Formatters](#formatters)
  1. [Padding](#padding)
  2. [Tables](#tables)
4. [Style Elements](#style-elements)
  1. [Built-In Elements](#built-in-elements)
  2. [Custom Elements](#custom-elements)

<h2 id="prompts">Prompts</h2>
Prompts are great for asking users for input beyond what is accepted by arguments.  For example, you might want to confirm with a user before doing an administrative task, or you might ask her to select from a list of possible choices.  Prompts accept `RDev\Console\Prompts\Question\IQuestion` objects.

<h4 id="confirmation">Confirmation</h4>
To ask a user to confirm an action with a simple "y" or "yes", use an `RDev\Console\Prompts\Questions\Confirmation`:

```php
use RDev\Console\Prompts\Prompt;
use RDev\Console\Prompts\Questions\Confirmation;
use RDev\Console\Responses\Formatters\PaddingFormatter;

$prompt = new Prompt(new PaddingFormatter());
// This will return true if the answer began with "y" or "Y"
$prompt->ask(new Confirmation("Are you sure you want to continue?"));
```

<h4 id="multiple-choice">Multiple Choice</h4>
Multiple choice questions are great for listing choices that might otherwise be difficult for a user to remember.  An `RDev\Console\Prompts\Questions\MultipleChoice` accepts question text and a list of choices:

```php
use RDev\Console\Prompts\Questions\MultipleChoice;

$choices = ["Boeing 747", "Boeing 757", "Boeing 787"];
$question = new MultipleChoice("Select your favorite airplane", $choices);
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

<h2 id="responses">Responses</h2>
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

<h2 id="formatters">Formatters</h2>
Formatters are great for nicely-formatting output to the console.

<h4 id="padding">Padding</h4>
The `RDev\Console\Responses\Formatters\PaddingFormatter` formatter allows you to create column-like output.  It accepts an array of column values.  The second parameter is a callback that will format each row's contents.  Let's look at an example:
 
```php
use RDev\Console\Responses\Formatters\PaddingFormatter;

$paddingFormatter = new PaddingFormatter();
$rows = [
    ["George", "Carlin", "great"],
    ["Chris", "Rock", "good"],
    ["Jim", "Gaffigan", "pale"]
];
$paddingFormatter->format($rows, function($row)
{
    return $row[0] . " - " . $row[1] . " - " . $row[2];
});
```

This will return:
```
George - Carlin   - great
Chris  - Rock     - good
Jim    - Gaffigan - pale
```

There are a few useful functions for customizing the padding formatter:

* `setEOLChar()`
  * Sets the end-of-line character
* `setPadAfter()`
  * Sets whether to pad before or after strings
* `setPaddingString()`
  * Sets the padding string

<h4 id="tables">Tables</h4>
ASCII tables are a great way to show tabular data in a console.  To create a table, use `RDev\Console\Responses\Formatters\TableFormatter`:

```php
use RDev\Console\Responses\Formatters\PaddingFormatter;
use RDev\Console\Responses\Formatters\TableFormatter;

$table = new TableFormatter(new PaddingFormatter());
$rows = [
    ["Sean", "Connery"],
    ["Pierce", "Brosnan"]
];
$table->format($rows);
```

This will return:

```
+--------+---------+
| Sean   | Connery |
| Pierce | Brosnan |
+--------+---------+
```

Headers can also be included in tables:

```php
$headers = ["First", "Last"];
$table->format($rows, $headers);
```

This will return:

```
+--------+---------+
| First  | Last    |
+--------+---------+
| Sean   | Connery |
| Pierce | Brosnan |
+--------+---------+
```

There are a few useful functions for customizing the look of tables:

* `setCellPaddingString()`
  * Sets the cell padding string
* `setEOLChar()`
  * Sets the end-of-line character
* `setHorizontalBorderChar()`
  * Sets the horizontal border character
* `setIntersectionChar()`
  * Sets the row/column intersection character
* `setPadAfter()`
  * Sets whether to pad before or after strings
* `setVerticalBorderChar()`
  * Sets the vertical border character

<h2 id="style-elements">Style Elements</h2>
RDev supports HTML-like style elements to perform basic output formatting like background color, foreground color, boldening, and underlining.  It does this by parsing the string into an **Abstract Syntax Tree**, and then converting each node in the tree into the appropriate ANSI codes.  For example, writing:

```
<b>Hello!</b>
```

...will output "<b>Hello!</b>".  You can even nest elements:

```
<u>Hello, <b>Dave</b></u>
```

..., which will output an underlined string where "Dave" is both bold AND underlined.

<h4 id="built-in-elements">Built-In Elements</h4>
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

<h4 id="custom-elements">Custom Elements</h4>
You can create your own style elements.  Elements are registered to `RDev\Console\Responses\Compilers\Compiler`.  To register a custom element, use a bootstrapper:

```php
namespace MyApp\Bootstrappers\Console;
use RDev\Applications\Bootstrappers\Bootstrapper;
use RDev\Console\Responses\Compilers\ICompiler;
use RDev\Console\Responses\Formatters\Elements\Colors;
use RDev\Console\Responses\Formatters\Elements\Element;
use RDev\Console\Responses\Formatters\Elements\Style;
use RDev\Console\Responses\Formatters\Elements\TextStyles;

class CustomElements extends Bootstrapper
{
    public function run(ICompiler $compiler)
    {
        $compiler->getElements()->add(
            new Element("foo", new Style(Colors::BLACK, Colors::YELLOW, [TextStyles::BOLD]))
        );
    }
}
```