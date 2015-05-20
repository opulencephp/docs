# Views

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Usage](#basic-usage)
3. [Caching](#caching)
  1. [Garbage Collection](#garbage-collection)
4. [Cross-Site Scripting](#cross-site-scripting)
5. [Extending Templates](#extending-templates)
  1. [Example](#example)
  2. [Parts](#parts)
  3. [Parents](#parents)
  4. [Difference Between Tags and Statements](#difference-between-tags-and-statements)
6. [Including Templates](#including-templates)
7. [Using PHP in Your Template](#using-php-in-your-template)
8. [Functions](#functions)
  1. [RDev Functions](#rdev-functions)
  2. [Custom Template Functions](#custom-template-functions)
  3. [Using Template Functions in PHP Code](#using-template-functions-in-php-code)
9. [Extending the Compiler](#extending-the-compiler)
10. [Escaping Tag Delimiters](#escaping-tag-delimiters)
11. [Custom Tag Delimiters](#custom-tag-delimiters)
12. [Template Factory](#template-factory)
  1. [Builders](#builders)
  2. [Aliasing](#aliasing)

<h2 id="introduction">Introduction</h2>
**RDev** has a template system, which is meant to simplify adding dynamic content to web pages.  You can inject data into your pages, create loops for generating iterative items, escape unsanitized text, and add your own tag extensions.  Unlike other popular template libraries out there, you can use plain old PHP for simple constructs such as if/else statements and loops.

<h2 id="basic-usage">Basic Usage</h2>
Templates hold raw content for pages and page parts.  In order to compile this raw content into finished templates, we `compile()` them using a compiler that implements `RDev\Views\Compilers\ICompiler` (`RDev\Views\Compilers\Compiler` come built-in to RDev).  By separating compiling into a separate class, we separate the concerns of templates and compiling templates, thus satisfying the *Single Responsibility Principle* (*SRP*).  Let's take a look at a basic example:

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

<h2 id="extending-templates">Extending Templates</h2>
Most templates extend some sort of master template.  To make your life easy, RDev builds support for this functionality into its templates.  RDev uses a *statement tag* `{% %}` for RDev-specific logic statements.  They provide the ability do such things as extend templates.

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

We created a *part* named "sidebar".  When the child gets compiled, the contents of that part will be shown in any `{% show() %}` statement whose parameter matches the name of the part. We will get the following:

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

<h4 id="difference-between-tags-and-statements">Difference Between Tags and Statements</h4>
You might be asking what the difference between tags and statements is.  Tags are temporary placeholders for data that is inserted through a controller.  Statements, on the other hand, provide a shorthand for executing logic entirely within a template.

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

<h2 id="functions">Functions</h2>
RDev supports using PHP functions in tags:

##### Template
```
Hello, {{strtoupper("Dave")}}
```

##### Output
```
Hello, DAVE
```

You can also pass variables into your functions in the template and set them using `setVar()`.  RDev even supports nested functions:

##### Template
```
{{trim(ucwords(" dave young "))}}
```

##### Output
```
Dave Young
```

<h4 id="rdev-functions">RDev Functions</h4>
RDev supplies some built-in functions:
* `charset()`
  * Returns HTML used to select a character set
  * Accepts the following arguments:
    1. `string $charset` - The character set to use
* `css()`
  * Returns HTML used to link to a CSS stylesheet
  * Accepts the following arguments:
    1. `array|string $paths` - The path or list of paths to the stylesheets
* `favicon()`
  * Returns HTML used to display a favicon
  * Accepts the following arguments:
    1. `string $path` - The path to the favicon image
* `httpEquiv()`
  * Returns HTML used to create an http-equiv attribute
  * Accepts the following arguments:
    1. `string $name` - The name of the http-equiv attribute, eg "refresh"
    2. `mixed $value` - The value of the attribute
* `metaDescription()`
  * Returns HTML used to display a meta description
  * Accepts the following arguments:
    1. `string $metaDescription` - The meta description to use
* `metaKeywords()`
  * Returns HTML used to display meta keywords
  * Accepts the following arguments:
    1. `array $metaKeywords` - The list of meta keywords to use
* `pageTitle()`
  * Returns HTML used to display a title
  * Accepts the following arguments:
    1. `string $title` - The title to use
* `route()`
  * Returns a URL that is created using the rules of the input route name
  * Accepts the following arguments:
    1. `string $routeName` - The name of the route whose URL we're creating
    2. `array|mixed $args` - The arguments to pass into the `URLGenerator` to fill any host or path variables in the route ([learn more about the `URLGenerator`](routing#url-generators))
* `script()`
  * Returns HTML used to link to a script file
  * Accepts the following arguments:
    1. `array|string $paths` - The path or list of paths to the scripts
    2. `string $type` - The script type, eg "text/javascript"

Since these functions output HTML, use them inside unescaped tags.  Here's an example of how to use these functions:

##### Template
```
<!DOCTYPE html>
<html>
    <head>
        {{!charset("utf-8")!}}
        {{!httpEquiv("content-type", "text/html")!}}
        {{!pageTitle("My Website")!}}
        {{!metaDescription("An example website")!}}
        {{!metaKeywords(["RDev", "sample"])!}}
        {{!favicon("favicon.ico")!}}
        {{!css("stylesheet.css")!}}
    </head>
    <body>
        Hello, World!
        {{!script(["jquery.js", "angular.js"])!}}
    </body>
</html>
```

##### Application Code
```php
$template->setContents(TEMPLATE);
echo $compiler->compile($template);
```

This will output:

```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta http-equiv="content-type" content="text/html">
        <title>My Website</title>
        <meta name="description" content="An example website">
        <meta name="keywords" content="RDev,sample">
        <link href="favicon.ico" rel="shortcut icon">
        <link href="stylesheet.css" rel="stylesheet">
    </head>
    <body>
        Hello, World!
        <script type="text/javascript" src="jquery.js"></script>
        <script type="text/javascript" src="angular.js"></script>
    </body>
</html>
```

<h4 id="custom-template-functions">Custom Template Functions</h4>
It's possible to add custom functions to your template.  For example, you might want to add a salutation to a last name in your template.  This salutation would need to know the last name, whether or not the person is a male, and if s/he is married.  You could set tags with the formatted value, but this would require a lot of duplicated formatting code in your application.  Instead, save yourself some work and register the function to the compiler:
##### Template
```
Hello, {{salutation("Young", false, true)}}
```

##### Application Code
```php
$template->setContents($fileSystem->read(PATH_TO_HTML_TEMPLATE));
// Our function simply needs to have a printable return value
$compiler->registerTemplateFunction("salutation", function($lastName, $isMale, $isMarried)
{
    if($isMale)
    {
        $salutation = "Mr.";
    }
    elseif($isMarried)
    {
        $salutation = "Mrs.";
    }
    else
    {
        $salutation = "Ms.";
    }

    return $salutation . " " . $lastName;
});
echo $compiler->compile($template); // "Hello, Mrs. Young"
```

<h4 id="using-template-functions-in-php-code">Using Template Functions in PHP Code</h4>
You may execute template functions in your PHP code by calling `RDev\Views\Compilers\ICompiler::executeTemplateFunction()`.  Let's take a look at an example that displays a pretty HTML page title formatted like `My Site | NAME_OF_PAGE`:

##### Template
```
<!DOCTYPE html>
<html>
    <head>
        {{!myPageTitle("About")!}}
    </head>
    <body>
        My About Page
    </body>
</html>
```

##### Application Code
```php
$compiler->registerTemplateFunction("myPageTitle", function($title) use ($compiler, $template)
{
    // Take advantage of the built-in template function
    return $compiler->executeTemplateFunction("pageTitle", [$template, $title . " | My Site"]);
});
$template->setContents(TEMPLATE);
echo $compiler->compile($template);
```

This will output:

```
<!DOCTYPE html>
<html>
    <head>
        <title>My Site | About</title>
    </head>
    <body>
        My About Page
    </body>
</html>
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

<h2 id="escaping-tag-delimiters">Escaping Tag Delimiters</h2>
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

<h2 id="custom-tag-delimiters">Custom Tag Delimiters</h2>
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

<h2 id="template-factory">Template Factory</h2>
Having to always pass in the full path to load a template from a file can get annoying.  It can also make it more difficult to switch your template directory should you ever decide to do so.  This is where a `Factory` comes in handy.  Simply pass in a `FileSystem` and the directory that your templates are stored in, and you'll never have to repeat yourself:
 
```php
use RDev\Files\FileSystem;
use RDev\Views\Factories\TemplateFactory;

$fileSystem = new FileSystem();
// Assume we keep all templates at "/var/www/html/views"
$factory = new TemplateFactory($fileSystem, "/var/www/html/views");
// This creates a template from "/var/www/html/views/login.html"
$loginTemplate = $factory->create("login.html");
// This creates a template from "/var/www/html/views/books/list.html"
$bookListTemplate = $factory->create("books/list.html");
```
 
> **Note:** Preceding slashes in `create()` are not necessary.
 
<h4 id="builders">Builders</h4>
 
Repetitive tasks such as setting up templates should not be done in controllers.  That should be left to dedicated classes called `Builders`.  A `Builder` is a class that does any setup on a template after it is created by the factory.  You can register a `Builder` to a template so that each time that template is loaded by the factory, the builders are run.  Register builders via `ITemplateFactory::registerBuilder()`.  The second parameter is a callback that returns an instance of your builder.  Builders are lazy-loaded (ie they're only created when they're needed), which is why a callback is passed instead of the actual instance.  Your builder classes must implement `RDev\Views\IBuilder`.  It's recommended that you register your builders via a [`Bootstrapper`](bootstrappers).

Let's take a look at an example:

```
<!-- Let's say this markup is in "Index.html" -->
<h1>{{siteName}}</h1>
{{content}}
```

```php
namespace MyApp\Builders;
use RDev\Files\FileSystem;
use RDev\Views\Factories\TemplateFactory;
use RDev\Views\IBuilder;
use RDev\Views\ITemplate;

class MyBuilder implements IBuilder
{
    public function build(ITemplate $template)
    {
        $template->setTag("siteName", "My Website");
        
        return $template;
    }
}

// Register our builder to "Index.html"
$factory = new TemplateFactory(new FileSystem(), __DIR__ . "/tmp");
$callback = function()
{
    return new MyBuilder();
};
$factory->registerBuilder("Index.html", $callback);

// Now, whenever we request "Index.html", the "siteName" tag will be set to "My Website"
$template = $factory->create("Index.html");
echo $template->getTag("siteName"); // "My Website"
```

<h4 id="aliasing">Aliasing</h4>
Multiple pages might use the same template, but with different tag and variable values.  This creates a problem if we want to register a builder for one page that shares a template with others.  We don't want to register that builder for all the other pages that share the template.  This is where `ITemplateFactory::alias()` comes in handy.  You can create an alias, and then register builders to that alias.  `ITemplateFactory::create()` accepts either a template path or an alias.

```php
$templateFactory->alias("Home", "Master.html");
$templateFactory->alias("About", "Master.html");
$templateFactory->registerBuilder("Master.html", function()
{
    return new MasterBuilder();
});
$templateFactory->registerBuilder("Home", function()
{
    return new HomeBuilder();
});
$templateFactory->registerBuilder("About", function()
{
    return new AboutBuilder();
});
$masterTemplate = $templateFactory->create("Master.html"); // MasterBuilder is run
$homeTemplate = $templateFactory->create("Home"); // MasterBuilder and HomeBuilder are run
$aboutTemplate = $templateFactory->create("About"); // MasterBuilder and AboutBuilder are run
```

> **Note:** Builders registered to the template that an alias refers to will be run as well as builders registered to the alias.