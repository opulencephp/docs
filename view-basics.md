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
4. [Using PHP in Your View](#using-php-in-your-view)
5. [Extending Views](#extending-views)
  1. [Example](#example)
  2. [Parts](#parts)
  3. [Parents](#parents)
6. [Including Views](#including-views)
7. [Caching](#caching)
  1. [Garbage Collection](#garbage-collection)
8. [Extending the Compiler](#extending-the-compiler)

<h2 id="introduction">Introduction</h2>
Opulence has a template system, which is meant to simplify adding dynamic content to web pages.  You can inject data into your pages, create loops for generating iterative items, escape unsanitized text, and add your own tag extensions.  Unlike other popular template libraries out there, you can use plain old PHP for simple constructs such as if/else statements and loops.

<h2 id="basic-usage">Basic Usage</h2>

<h4 id="tags">Tags</h4>
Tags are placeholders for values in your views.  For example, let's say that you want to print `Hello, {{$username}}` at the top of your website.  To set the "username" variable, use `setVar()`:

```php
$view->setVar("username", "Dave");
```

<h4 id="statements">Statements</h4>
Statements perform view logic.  For example, they can be used to denote a [view that extends another view](#extending-views) or [include another view](#including-views).

> **Note:** You might be asking what the difference between tags and statements is.  Tags are temporary placeholders for data that is inserted through a controller.  Statements, on the other hand, provide a shorthand for executing logic entirely within a view.

<h4 id="escaping-tag-delimiters">Escaping Tag Delimiters</h4>
Want to escape a tag delimiter?  Easy!  Just add a backslash before the opening tag like so:
##### View
```
Hello, {{$username}}.  \{{I am escaped}}! \{{!Me too!}}. \<%So am I%>.
```
##### Application Code
```php
$view->setContents($fileSystem->read(PATH_TO_HTML_VIEW));
$view->setVar("username", "Mr Schwarzenegger");
echo $compiler->compile($view); // "Hello, Mr Schwarzenegger.  {{I am escaped}}! {{!Me too!}}. <%So am I%>."
```

<h4 id="custom-tag-delimiters">Custom Tag Delimiters</h4>
Want to use a custom character/string for the tag delimiters?  Easy!  Just specify it in the `View` object like so:
##### View
```
^^name$$ ++food--
```
##### Application Code
```php
$view->setContents($fileSystem->read(PATH_TO_HTML_VIEW));
$view->setDelimiters($view::DELIMITER_TYPE_ESCAPED_TAG, ["^^", "$$"]);
// You can also override the unescaped tag delimiters
$view->setDelimiters($view::DELIMITER_TYPE_UNESCAPED_TAG, ["++", "--"]);
// You can even override statement delimiters
$view->setDelimiters($view::DELIMITER_TYPE_STATEMENT, ["(*", "*)"]);
// Try setting some tags
$view->setVar("name", "A&W");
$view->setVar("food", "Root Beer");
echo $compiler->compile($view); // "A&amp;W Root Beer"
```

<h4 id="compiling">Compiling</h4>
Opulence compiles views using a compiler that implements `Opulence\Views\Compilers\ICompiler` (`Opulence\Views\Compilers\Compiler` come built-into Opulence).  By separating compiling into a separate class, we separate the concerns of views and compiling views, thus satisfying the **Single Responsibility Principle** (**SRP**).  Let's take a look at a basic example:

##### View
```
Hello, {{$username}}
```
##### Application Code
```php
use Opulence\Files\FileSystem;
use Opulence\Views\Caching\Cache;
use Opulence\Views\Compilers\Compiler;
use Opulence\Views\Factories\ViewFactory;
use Opulence\Views\Filters\XSSFilter;
use Opulence\Views\View;

$fileSystem = new FileSystem();
$cache = new Cache($fileSystem, "/tmp");
$viewFactory = new ViewFactory($fileSystem, PATH_TO_VIEWS);
$xssFilter = new XSSFilter();
$compiler = new Compiler($cache, $viewFactory, $xssFilter);
$view = new View();
$view->setContents($fileSystem->read(PATH_TO_HTML_VIEW));
$view->setVar("username", "Dave");
echo $compiler->compile($view); // "Hello, Dave"
```

Alternatively, you could just pass in a view's contents to its constructor:
```php
use Opulence\Files\FileSystem;
use Opulence\Views\Caching\Cache;
use Opulence\Views\Compilers\Compiler;
use Opulence\Views\Factories\ViewFactory;
use Opulence\Views\Filters\XSSFilter;
use Opulence\Views\View;

$fileSystem = new FileSystem();
$cache = new Cache($fileSystem, "/tmp");
$viewFactory = new ViewFactory($fileSystem, PATH_TO_VIEWS);
$xssFilter = new XSSFilter();
$compiler = new Compiler($cache, $viewFactory, $xssFilter);
$view = new View('Hello, {{$username}}');
$view->setVar("username", "Dave");
echo $compiler->compile($view); // "Hello, Dave"
```

<h2 id="cross-site-scripting">Cross-Site Scripting</h2>
Tags are automatically sanitized to prevent cross-site scripting (XSS) when using the "{{" and "}}" tags.  To display unescaped data, simply use "{{!MY_UNESCAPED_TAG_NAME_HERE!}}".
##### View
```
{{$name}} vs {{!$name!}}
```
##### Application Code
```php
$view->setContents($fileSystem->read(PATH_TO_HTML_VIEW));
$view->setVar("name", "A&W");
echo $compiler->compile($view); // "A&amp;W vs A&W"
```

Alternatively, you can output a string literal inside tags:
##### View
```
{{"A&W"}} vs {{!"A&W"!}}
```

This will output "A&amp;amp;W vs A&amp;W".

<h2 id="using-php-in-your-view">Using PHP in Your View</h2>
Keeping your view separate from your business logic is important.  However, there are times when it would be nice to be able to execute some PHP code to do things like for() loops to output a list.  There is no need to memorize library-specific constructs here.  With Opulence's template system, you can do this:
##### View
```
<ul><?php foreach(["foo", "bar"] as $item): ?>
    <li>{{$item}}</li>
<?php endforeach; ?></ul>
```
##### Application Code
```php
$view->setContents($fileSystem->read(PATH_TO_HTML_VIEW));
echo $compiler->compile($view); // "<ul><li>foo</li><li>bar</li></ul>"
```

You can also inject values from your application code into variables in your view:
##### View
```
<?php if($isAdministrator): ?>
Hello, Administrator
<?php endif; ?>
```
##### Application Code
```php
$view->setContents($fileSystem->read(PATH_TO_HTML_VIEW));
$view->setVar("isAdministrator", true);
echo $compiler->compile($view); // "Hello, Administrator"
```

> **Note:** It's recommended to keep as much business logic out of the views as you can.  In other words, utilize PHP in the view to simplify things like lists or basic if/else statements or loops.  Perform the bulk of the logic in the application code, and inject data into the view when necessary.

<h2 id="extending-views">Extending Views</h2>
Most views extend some sort of master view.  To make your life easy, Opulence builds support for this functionality into its views.  Opulence uses a statement tag `<% %>` for Opulence-specific logic statements.  They provide the ability do such things as extend views.

<h4 id="example">Example</h4>

##### Master.html
```
Hello, world!
```

##### Child
```
<% extends("Master.html") %>
Hello, Dave!
```

When the child view gets compiled, the `Master.html` view is automatically created by an `Opulence\Views\Factories\IViewFactory` and inserted into the view to produce the following output:

```
Hello, world!
Hello, Dave!
```

> **Note:** When extending a view, the child view inherits all of the parent's parts, tags, and variable values.  If A extends B, which extends C, tags/parts/variables from part B will overwrite any identically-named tags/parts/variables from part C.

<h4 id="parts">Parts</h4>
Another common case is a master view that is leaving a child view to fill in some information.  For example, let's say our master has a sidebar, and we want to define the sidebar's contents in the child view.  Use the `<% show("NAME_OF_PART") %>` statement:

##### Master.html
```
<div id="sidebar">
    <% show("sidebar") %>
</div>
```

##### Child
```
<% extends("Master.html") %>
<% part("sidebar") %>
<ul>
    <li><a href="/">Home</a></li>
    <li><a href="/about">About</a></li>
</ul>
<% endpart %>
```

We created a part named "sidebar".  When the child gets compiled, the contents of that part will be shown in any `<% show() %>` statement whose parameter matches the name of the part. We will get the following:

```
<div id="sidebar">
    <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/about">About</a></li>
    </ul>
</div>
```

<h4 id="parents">Parents</h4>
Sometimes, you'll want to add to a parent view's part.  To do so, use the `<% parent("NAME_OF_PART") %>` statement:

##### Master.html
```
<% part("greeting") %>
Hello
<% endpart %>
```

##### Child
```
<% extends("Master.html") %>
<% part("greeting") %>
<% parent("greeting") %>, world!
<% endpart %>
```

This will get compiled down to:

```
Hello, world!
```

<h2 id="including-views">Including Views</h2>
Including another view (in much the same way PHP's `include` works) is an easy way to not repeat yourself.  Here's an example of how to include a view:

##### IncludedView.html
```
Hello, world!
```

##### Master View
```
<div id="important-message">
    <% include("IncludedView.html") %>
</div>
```

This will compile to:

```
<div id="important-message">
    Hello, world!
</div>
```

<h2 id="caching">Caching</h2>
To improve the speed of view compiling, views are cached using a class that implements `Opulence\Views\Caching\ICache` (`Opulence\Views\Caching\Cache` comes built-in to Opulence).  You can specify how long a view should live in cache using `setLifetime()`.  If you do not want views to live in cache at all, you can specify a non-positive lifetime.  If you'd like to create your own cache engine for views, just implement `ICache` and pass it into your `View` class.

<h4 id="garbage-collection">Garbage Collection</h4>
Occasionally, you should clear out old cached view files to save disk space.  If you'd like to call it explicitly, call `gc()` on your cache object.  `Cache` has a mechanism for performing this garbage collection every so often.  You can customize how frequently garbage collection is run:
 
```php
use Opulence\Files\FileSystem;
use Opulence\Views\Caching\Cache;

// Make 123 out of every 1,000 view compilations trigger garbage collection
$cache = new Cache(new FileSystem(), "/tmp", 123, 1000);
```
Or use `setGCChance()`:
```php
// Make 1 out of every 500 view compilations trigger garbage collection
$cache->setGCChance(1, 500);
```

<h2 id="extending-the-compiler">Extending the Compiler</h2>
Let's pretend that there's some unique feature or syntax you want to implement in your view that cannot currently be compiled with Opulence's `Compiler`.  Using `Compiler::registerSubCompiler()`, you can compile the syntax in your view to the desired output.  Opulence itself uses `registerSubCompiler()` to compile statements, PHP, and tags in views.

Let's take a look at what should be passed into `registerSubCompiler()`:

  1. `Opulence\Views\Compilers\SubCompilers\ISubCompiler $subCompiler`
  2. `int|null $priority`
    * If your sub-compiler needs to be executed before other compilers, simply pass in an integer to prioritize the sub-compiler (1 is the highest)
    * If you do not specify a priority, then the compiler will be executed after the prioritized sub-compilers in the order it was added

Let's take a look at an example that converts HTML comments to an HTML list of those comments:

```php
use Opulence\Views\Compilers\SubCompilers\ISubCompiler;
use Opulence\Views\IView;

class MySubCompiler implements ISubCompiler
{
    public function compile(IView $view, $content)
    {
        return "<ul>" . preg_replace("/<!--((?:(?!-->).)*)-->/", "<li>$1</li>", $content) . "</ul>";
    }
}

$compiler->registerSubCompiler(new MySubCompiler());
$view->setContents("<!--Comment 1--><!--Comment 2-->");
echo $compiler->compile($view); // "<ul><li>Comment 1</li><li>Comment 2</li></ul>"
```