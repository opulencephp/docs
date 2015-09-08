# Fortune

## Table of Contents
1. [Introduction](#introduction)
2. [Tags](#tags)
  1. [Sanitized Tags](#sanitized-tags)
  2. [Unsanitized Tags](#unsanitized-tags)
3. [Directives](#directives)
  1. [If Statements](#if-statements)
  2. [Loops](#loops)
  3. [Including Views](#including-views)
  4. [Extending Views](#extending-views)
      1. [Parents](#parents)
  5. [Parts](#parts)
  6. [Creating Directives](#creating-directives)
4. [Functions](#functions)
  1. [Fortune Functions](#fortune-functions)
  2. [Custom Functions](#custom-functions)
  3. [Using View Functions in PHP Code](#using-view-functions-in-php-code)
5. [Delimiters](#delimiters)
  1. [Escaping Delimiters](#escaping-delimiters)
  2. [Changing Delimiters](#changing-delimiters)
6. [Caching](#caching)
  1. [Garbage Collection](#garbage-collection)

<h2 id="introduction">Introduction</h2>
Fortune is the view engine that comes built into Opulence.  It simplifies adding dynamic content to web pages.  You can inject data into your pages, extend other views, prevent XSS attacks, and even extend the compiler.  To get started using Fortune, simply create files with the extension "fortune", eg `Master.fortune`.  Opulence will detect that the file is a Fortune template and will use the Fortune compiler.

Fortune uses a lexer to tokenize the raw template into a stream of tokens.  The tokens are then parsed into an abstract syntax tree, which is transpiled into regular PHP code.  The transpiler caches the generated PHP code to significantly speed up subsequent requests.

<h2 id="tags">Tags</h2>
Tags are syntactic sugar for echoing PHP data in your view.  You can put any printable value inside of tags, including PHP code:

```
{{"foo"}}
{{$bar}}
{{isset($baz) ? "Baz is set" : "Baz is NOT set"}}
```
  
<h4 id="sanitized-tags">Sanitized Tags</h4>
Sanitized tags prevent malicious users from executing a cross-site scripting (XSS) attack:

```
{{"A&W > water"}}
```

This will compile to `A&amp;W &gt; water`.

<h4 id="unsanitized-tags">Unsanitized Tags</h4>
Unsanitized tags are useful for outputting HTML:

```
{{!"<title>My Title</title>"!}}
```

This will compile to `<title>My Title</title>`.

> **Note**:  Always sanitize user input before displaying it on your pages.

<h2 id="directives">Directives</h2>
Directives perform view logic.  For example, they can be used to denote a [view that extends another view](#extending-views) or [include another view](#including-views).

> **Note:** You might be asking what the difference between tags and directives is.  Tags are temporary placeholders for data that is inserted through a controller.  Directives, on the other hand, provide a shorthand for executing logic entirely within a view.

<h4 id="if-statements">If Statements</h4>
```php
<% if($user->isAdmin()) %>
    <a href="edit">Edit Post</a>
<% elseif($user->canViewPosts()) %>
    <a href="view">View Post</a>
<% else %>
    You cannot view this post
<% endif %>
```

<h4 id="loops">Loops</h4>
```
<% for($i = 1;$i <= 5;$i++) %>
    {{$i}}
<% endfor %>

<% foreach($posts as $post) %>
    <a href="posts/{{$post->getId()}}">{{$post->getTitle()}}</a>
<% endforeach %>

// If there are posts in $posts, display them
// Otherwise, display "There are no posts"
<% forif($posts as $post) %>
    <a href="posts/{{$post->getId()}}">{{$post->getTitle()}}</a>
<% forelse %>
    There are no posts
<% endif %>

<% while(true) %>
    Still looping
<% endwhile %>
```

<h4 id="including-views">Including Views</h4>
Including another view (in much the same way PHP's `include` works) is an easy way to not repeat yourself.  Here's an example of how to include a view:

##### Included.fortune
```
Hello, world!
```

##### Master.fortune
```
<div id="important-message">
    <% include("Included.fortune") %>
</div>
```

This will compile to:

```
<div id="important-message">
    Hello, world!
</div>
```

<h4 id="extending-views">Extending Views</h4>
Most views extend some sort of master view.  To make your life easy, Fortune builds support for this functionality into its views.

<h5 id="example">Example</h5>

##### Master.fortune
```
Hello, world!
```

##### Child.fortune
```
<% extends("Master.fortune") %>
Hello, Dave!
```

When the child view gets compiled, the `Master.fortune` view is automatically created by an `Opulence\Views\Factories\IViewFactory` and inserted into the view to produce the following output:

```
Hello, world!
Hello, Dave!
```

> **Note:** When extending a view, the child view inherits all of the parent's parts and variable values.  If A extends B, which extends C, parts and variables from part B will overwrite any identically-named parts and variables from part C.

<h5 id="parents">Parents</h5>
Sometimes, you'll want to add to a parent view's part.  To do so, use the `<% parent %>` directive:

##### Master.fortune
```
<% part("greeting") %>
Hello
<% endpart %>
```

##### Child.fortune
```
<% extends("Master.fortune") %>
<% part("greeting") %>
    <% parent %>, world!
<% endpart %>
```

This will get compiled down to:

```
Hello, world!
```

<h4 id="parts">Parts</h4>
Another common case is a master view that is leaving a child view to fill in some information.  For example, let's say our master has a sidebar, and we want to define the sidebar's contents in the child view.  Use the `<% show("NAME_OF_PART") %>` directive:

##### Master.fortune
```
<div id="sidebar">
    <% show("sidebar") %>
</div>
```

##### Child.fortune
```
<% extends("Master.fortune") %>
<% part("sidebar") %>
<ul>
    <li><a href="/">Home</a></li>
    <li><a href="/about">About</a></li>
</ul>
<% endpart %>
```

We created a part named "sidebar".  When the child gets compiled, the contents of that part will be shown in any `<% show() %>` directive whose parameter matches the name of the part. We will get the following:

```
<div id="sidebar">
    <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/about">About</a></li>
    </ul>
</div>
```

<h4 id="creating-directives">Creating Directives</h4>
To create your own Fortune directives, simply register them to the Fortune transpiler using `registerDirectiveTranspiler()`.  The first argument is the name of the directive, and the second is a callback that returns transpiled PHP code.  The callback optionally accepts an expression, which can be used when transpiling to PHP.  Registering your directive transpiler is most easily accomplished in a `Bootstrapper`.  Let's take a look at Fortune's `if` statement directive transpiler:

```php
use Opulence\Applications\Bootstrappers\Bootstrapper;
use Opulence\Views\Compilers\Fortune\ITranspiler;

class MyDirectives extends Bootstrapper
{
    public function run(ITranspiler $transpiler)
    {
        $transpiler->registerDirectiveTranspiler("if", function($expression)
        {
            // The expression will contain the surrounding parentheses 
            return '<?php if' . $expression . ': ?>';
        });
    }
}
```

If in our template we have `<% if($user->isAdmin()) %>`, Fortune will transpile it to `<?php if($user->isAdmin()): ?>`.

<h2 id="functions">Functions</h2>
Fortune supports using functions in views.  They are a great way to reuse logic or formatting throughout your views.  You can use PHP functions, functions defined by Fortune, or your very own functions.  For example, `{{strtoupper("Dave")}}` will output `DAVE`.

<h4 id="fortune-functions">Fortune Functions</h4>
Fortune supplies some built-in functions in the view library:
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
* `script()`
  * Returns HTML used to link to a script file
  * Accepts the following arguments:
    1. `array|string $paths` - The path or list of paths to the scripts
    2. `string $type` - The script type, eg "text/javascript"
    
If you're using the <a href="https://github.com/opulencephp/Project" target="_blank">skeleton project</a>, Fortune also supplies the following functions:
* `csrfInput()`
  * Returns a hidden input containing the CSRF token
* `csrfToken()`
  * Returns the current CSRF token
* `route()`
  * Returns a URL that is created using the rules of the input route name
  * Accepts the following arguments:
    1. `string $routeName` - The name of the route whose URL we're creating
    2. `array|mixed $args` - The arguments to pass into the `URLGenerator` to fill any host or path variables in the route ([learn more about the `URLGenerator`](routing#url-generators))

Since these functions output HTML, use them inside unsanitized tags.  Here's an example of how to use these functions:

##### View.fortune
```
<!DOCTYPE html>
<html>
    <head>
        {{!charset("utf-8")!}}
        {{!httpEquiv("content-type", "text/html")!}}
        {{!pageTitle("My Website")!}}
        {{!metaDescription("An example website")!}}
        {{!metaKeywords(["Opulence", "sample"])!}}
        {{!favicon("favicon.ico")!}}
        {{!css("stylesheet.css")!}}
    </head>
    <body>
        Hello, World!
        {{!script(["jquery.js", "angular.js"])!}}
    </body>
</html>
```

This will be compiled to:

```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta http-equiv="content-type" content="text/html">
        <title>My Website</title>
        <meta name="description" content="An example website">
        <meta name="keywords" content="Opulence,sample">
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

<h4 id="custom-functions">Custom Functions</h4>
It's possible to add custom functions to your view.  For example, you might want to add a salutation to a last name in your view.  This salutation would need to know the last name, whether or not the person is a male, and if s/he is married.  You could set tags with the formatted value, but this would require a lot of duplicated formatting code in your application.  Instead, save yourself some work and register the function to the Fortune transpiler.  This is most easily done in a `Bootstrapper`:

```php
use Opulence\Applications\Bootstrappers\Bootstrapper;
use Opulence\Views\Compilers\Fortune\ITranspiler;

class MyFunctions extends Bootstrapper
{
    public function run(ITranspiler $transpiler)
    {
        // Our function simply needs to have a printable return value
        $transpiler->registerViewFunction("salutation", function($lastName, $isMale, $isMarried)
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
    }
}
```

Compiling `Hello, {{salutation("Young", false, true)}}` will give us `Hello, Mrs. Young`.

<h4 id="using-view-functions-in-php-code">Using View Functions in PHP Code</h4>
You may call view functions in your PHP code by calling `Opulence\Views\Compilers\Fortune\ITranspiler::callViewFunction()`.  Let's take a look at an example that displays a pretty HTML page title formatted like `My Site | NAME_OF_PAGE`:

```php
$transpiler->registerViewFunction("myPageTitle", function($title) use ($transpiler, $view)
{
    // Take advantage of the built-in view function
    return $transpiler->callViewFunction("pageTitle", $view, $title . " | My Site");
});
```

Compiling `{{!myPageTitle("About")!}}` will give us `<title>About | My Site</title>`.

<h2 id="delimiters">Delimiters</h2>
Fortune allows you to customize the delimiters used with tags and directives.

<h4 id="escaping-delimiters">Escaping Delimiters</h4>
Lots of JavaScript frameworks use similar syntax to Fortune to display data.  To display a raw tag or directive, escape them using the `\` character:

```
\<% foo %>
\{{bar}}
```

This will compile to:

```
<% foo %>
{{bar}}
```

<h4 id="changing-delimiters">Changing Delimiters</h4>
Sometimes, it might just be easier to change the tag and/or directive delimiters used in a view.  To do so, use the `setDelimiters()` method on your view:
 
```php
use Opulence\Views\View;

$view = new View();
// Set new open and close delimiters for directives
$view->setDelimiters(View::DELIMITER_TYPE_DIRECTIVE, ["{%", "%}"]);
// Set new open and close delimiters for sanitized tags
$view->setDelimiters(View::DELIMITER_TYPE_SANITIZED_TAG, ["^^", "$$"]);
// Set new open and close delimiters for unsanitized tags
$view->setDelimiters(View::DELIMITER_TYPE_UNSANITIZED_TAG, ["++", "--"]);
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