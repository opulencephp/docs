# View Basics

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Usage](#basic-usage)
  1. [Tags](#tags)
  2. [Statements](#statements)
  3. [Escaping Tag Delimiters](#escaping-tag-delimiters)
  4. [Custom Tag Delimiters](#custom-tag-delimiters)
  5. [Compiling](#compiling)
3. [Cross-Site Scripting](#cross-site-scripting)
4. [Using PHP in Your Template](#using-php-in-your-template)
5. [Extending Templates](#extending-templates)
  1. [Example](#example)
  2. [Parts](#parts)
  3. [Parents](#parents)
6. [Including Templates](#including-templates)
7. [Caching](#caching)
  1. [Garbage Collection](#garbage-collection)
8. [Extending the Compiler](#extending-the-compiler)

<h2 id="introduction">Introduction</h2>
RDev has a template system, which is meant to simplify adding dynamic content to web pages.  You can inject data into your pages, create loops for generating iterative items, escape unsanitized text, and add your own tag extensions.  Unlike other popular template libraries out there, you can use plain old PHP for simple constructs such as if/else statements and loops.

<h2 id="basic-usage">Basic Usage</h2>

<h4 id="tags">Tags</h4>
Tags are placeholders for values in your templates.  For example, let's say that you want to print `Hello, {{username}}` at the top of your website.  To set the "username" tag, use `setTag()`:

```php
$template->setTag("username", "Dave");
```

<h4 id="statements">Statements</h4>
Statements perform template logic.  For example, they can be used to denote a [template that extends another template](#extending-templates) or [include another template](#including-templates).

> **Note:** You might be asking what the difference between tags and statements is.  Tags are temporary placeholders for data that is inserted through a controller.  Statements, on the other hand, provide a shorthand for executing logic entirely within a template.

<h4 id="escaping-tag-delimiters">Escaping Tag Delimiters</h4>
Want to escape a tag delimiter?  Easy!  Just add a backslash before the opening tag like so:
##### Template
```
Hello, {{username}}.  \{{I am escaped}}! \{{!Me too!}}. \{%So am I%}.
```
##### Application Code
```php
$template->setContents($fileSystem->read(PATH_TO_HTML_TEMPLATE));
$template->setTag("username", "Mr Schwarzenegger");
echo $compiler->compile($template); // "Hello, Mr Schwarzenegger.  {{I am escaped}}! {{!Me too!}}. {%So am I%}."
```

<h4 id="custom-tag-delimiters">Custom Tag Delimiters</h4>
Want to use a custom character/string for the tag delimiters?  Easy!  Just specify it in the `Template` object like so:
##### Template
```
^^name$$ ++food--
```
##### Application Code
```php
$template->setContents($fileSystem->read(PATH_TO_HTML_TEMPLATE));
$template->setDelimiters($template::DELIMITER_TYPE_ESCAPED_TAG, ["^^", "$$"]);
// You can also override the unescaped tag delimiters
$template->setDelimiters($template::DELIMITER_TYPE_UNESCAPED_TAG, ["++", "--"]);
// You can even override statement delimiters
$template->setDelimiters($template::DELIMITER_TYPE_STATEMENT, ["(*", "*)"]);
// Try setting some tags
$template->setTag("name", "A&W");
$template->setTag("food", "Root Beer");
echo $compiler->compile($template); // "A&amp;W Root Beer"
```

<h4 id="compiling">Compiling</h4>
RDev compiles templates using a compiler that implements `RDev\Views\Compilers\ICompiler` (`RDev\Views\Compilers\Compiler` come built-into RDev).  By separating compiling into a separate class, we separate the concerns of templates and compiling templates, thus satisfying the **Single Responsibility Principle** (**SRP**).  Let's take a look at a basic example:

##### Template
```
Hello, {{username}}
```
##### Application Code
```php
use RDev\Files\FileSystem;
use RDev\Views\Caching\Cache;
use RDev\Views\Compilers\Compiler;
use RDev\Views\Factories\TemplateFactory;
use RDev\Views\Filters\XSSFilter;
use RDev\Views\Template;

$fileSystem = new FileSystem();
$cache = new Cache($fileSystem, "/tmp");
$templateFactory = new TemplateFactory($fileSystem, PATH_TO_TEMPLATES);
$xssFilter = new XSSFilter();
$compiler = new Compiler($cache, $templateFactory, $xssFilter);
$template = new Template();
$template->setContents($fileSystem->read(PATH_TO_HTML_TEMPLATE));
$template->setTag("username", "Dave");
echo $compiler->compile($template); // "Hello, Dave"
```

Alternatively, you could just pass in a template's contents to its constructor:
```php
use RDev\Files\FileSystem;
use RDev\Views\Caching\Cache;
use RDev\Views\Compilers\Compiler;
use RDev\Views\Factories\TemplateFactory;
use RDev\Views\Filters\XSSFilter;
use RDev\Views\Template;

$fileSystem = new FileSystem();
$cache = new Cache($fileSystem, "/tmp");
$templateFactory = new TemplateFactory($fileSystem, PATH_TO_TEMPLATES);
$xssFilter = new XSSFilter();
$compiler = new Compiler($cache, $templateFactory, $xssFilter);
$template = new Template("Hello, {{username}}");
$template->setTag("username", "Dave");
echo $compiler->compile($template); // "Hello, Dave"
```

<h2 id="cross-site-scripting">Cross-Site Scripting</h2>
Tags are automatically sanitized to prevent cross-site scripting (XSS) when using the "{{" and "}}" tags.  To display unescaped data, simply use "{{!MY_UNESCAPED_TAG_NAME_HERE!}}".
##### Template
```
{{name}} vs {{!name!}}
```
##### Application Code
```php
$template->setContents($fileSystem->read(PATH_TO_HTML_TEMPLATE));
$template->setTag("name", "A&W");
echo $compiler->compile($template); // "A&amp;W vs A&W"
```

Alternatively, you can output a string literal inside tags:
##### Template
```
{{"A&W"}} vs {{!"A&W"!}}
```

This will output "A&amp;amp;W vs A&amp;W".

<h2 id="using-php-in-your-template">Using PHP in Your Template</h2>
Keeping your view separate from your business logic is important.  However, there are times when it would be nice to be able to execute some PHP code to do things like for() loops to output a list.  There is no need to memorize library-specific constructs here.  With RDev's template system, you can do this:
##### Template
```
<ul><?php foreach(["foo", "bar"] as $item): ?>
    <li>{{$item}}</li>
<?php endforeach; ?></ul>
```
##### Application Code
```php
$template->setContents($fileSystem->read(PATH_TO_HTML_TEMPLATE));
echo $compiler->compile($template); // "<ul><li>foo</li><li>bar</li></ul>"
```

You can also inject values from your application code into variables in your template:
##### Template
```
<?php if($isAdministrator): ?>
Hello, Administrator
<?php endif; ?>
```
##### Application Code
```php
$template->setContents($fileSystem->read(PATH_TO_HTML_TEMPLATE));
$template->setVar("isAdministrator", true);
echo $compiler->compile($template); // "Hello, Administrator"
```

> **Note:** It's recommended to keep as much business logic out of the templates as you can.  In other words, utilize PHP in the template to simplify things like lists or basic if/else statements or loops.  Perform the bulk of the logic in the application code, and inject data into the template when necessary.

<h2 id="extending-templates">Extending Templates</h2>
Most templates extend some sort of master template.  To make your life easy, RDev builds support for this functionality into its templates.  RDev uses a statement tag `{% %}` for RDev-specific logic statements.  They provide the ability do such things as extend templates.

<h4 id="example">Example</h4>

##### Master.html
```
Hello, world!
```

##### Child
```
{% extends("Master.html") %}
Hello, Dave!
```

When the child template gets compiled, the `Master.html` template is automatically created by an `RDev\Views\Factories\ITemplateFactory` and inserted into the template to produce the following output:

```
Hello, world!
Hello, Dave!
```

> **Note:** When extending a template, the child template inherits all of the parent's parts, tags, and variable values.  If A extends B, which extends C, tags/parts/variables from part B will overwrite any identically-named tags/parts/variables from part C.

<h4 id="parts">Parts</h4>
Another common case is a master template that is leaving a child template to fill in some information.  For example, let's say our master has a sidebar, and we want to define the sidebar's contents in the child template.  Use the `{% show("NAME_OF_PART") %}` statement:

##### Master.html
```
<div id="sidebar">
    {% show("sidebar") %}
</div>
```

##### Child
```
{% extends("Master.html") %}
{% part("sidebar") %}
<ul>
    <li><a href="/">Home</a></li>
    <li><a href="/about">About</a></li>
</ul>
{% endpart %}
```

We created a part named "sidebar".  When the child gets compiled, the contents of that part will be shown in any `{% show() %}` statement whose parameter matches the name of the part. We will get the following:

```
<div id="sidebar">
    <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/about">About</a></li>
    </ul>
</div>
```

<h4 id="parents">Parents</h4>
Sometimes, you'll want to add to a parent template's part.  To do so, use the `{% parent("NAME_OF_PART") %}` statement:

##### Master.html
```
{% part("greeting") %}
Hello
{% endpart %}
```

##### Child
```
{% extends("Master.html") %}
{% part("greeting") %}
{% parent("greeting") %}, world!
{% endpart %}
```

This will get compiled down to:

```
Hello, world!
```

<h2 id="including-templates">Including Templates</h2>
Including another template (in much the same way PHP's `include` works) is an easy way to not repeat yourself.  Here's an example of how to include a template:

##### IncludedTemplate.html
```
Hello, world!
```

##### Master Template
```
<div id="important-message">
    {% include("IncludedTemplate.html") %}
</div>
```

This will compile to:

```
<div id="important-message">
    Hello, world!
</div>
```

<h2 id="caching">Caching</h2>
To improve the speed of template compiling, templates are cached using a class that implements `RDev\Views\Caching\ICache` (`RDev\Views\Caching\Cache` comes built-in to RDev).  You can specify how long a template should live in cache using `setLifetime()`.  If you do not want templates to live in cache at all, you can specify a non-positive lifetime.  If you'd like to create your own cache engine for templates, just implement `ICache` and pass it into your `Template` class.

<h4 id="garbage-collection">Garbage Collection</h4>
Occasionally, you should clear out old cached template files to save disk space.  If you'd like to call it explicitly, call `gc()` on your cache object.  `Cache` has a mechanism for performing this garbage collection every so often.  You can customize how frequently garbage collection is run:
 
```php
use RDev\Files\FileSystem;
use RDev\Views\Caching\Cache;

// Make 123 out of every 1,000 template compilations trigger garbage collection
$cache = new Cache(new FileSystem(), "/tmp", 123, 1000);
```
Or use `setGCChance()`:
```php
// Make 1 out of every 500 template compilations trigger garbage collection
$cache->setGCChance(1, 500);
```

<h2 id="extending-the-compiler">Extending the Compiler</h2>
Let's pretend that there's some unique feature or syntax you want to implement in your template that cannot currently be compiled with RDev's `Compiler`.  Using `Compiler::registerSubCompiler()`, you can compile the syntax in your template to the desired output.  RDev itself uses `registerSubCompiler()` to compile statements, PHP, and tags in templates.

Let's take a look at what should be passed into `registerSubCompiler()`:

  1. `RDev\Views\Compilers\SubCompilers\ISubCompiler $subCompiler`
  2. `int|null $priority`
    * If your sub-compiler needs to be executed before other compilers, simply pass in an integer to prioritize the sub-compiler (1 is the highest)
    * If you do not specify a priority, then the compiler will be executed after the prioritized sub-compilers in the order it was added

Let's take a look at an example that converts HTML comments to an HTML list of those comments:

```php
use RDev\Views\Compilers\SubCompilers\ISubCompiler;
use RDev\Views\ITemplate;

class MySubCompiler implements ISubCompiler
{
    public function compile(ITemplate $template, $content)
    {
        return "<ul>" . preg_replace("/<!--((?:(?!-->).)*)-->/", "<li>$1</li>", $content) . "</ul>";
    }
}

$compiler->registerSubCompiler(new MySubCompiler());
$template->setContents("<!--Comment 1--><!--Comment 2-->");
echo $compiler->compile($template); // "<ul><li>Comment 1</li><li>Comment 2</li></ul>"
```