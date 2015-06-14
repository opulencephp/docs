# View Functions

## Table of Contents
1. [Introduction](#introduction)
2. [PHP Functions](#php-functions)
3. [RDev Functions](#rdev-functions)
4. [Custom Functions](#custom-functions)
5. [Using Template Functions in PHP Code](#using-template-functions-in-php-code)

<h2 id="introduction">Introduction</h2>
RDev supports using functions in templates.  They are a great way to reuse logic or formatting throughout your templates.  You can use PHP functions, functions defined by RDev, or your very own functions.

<h2 id="php-functions">PHP Functions</h2>
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

<h2 id="rdev-functions">RDev Functions</h2>
RDev supplies some built-in functions in the view library:
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
    
If you're using the <a href="https://github.com/ramblingsofadev/Project" target="_blank">skeleton project</a>, RDev also supplies the following functions:
* `csrfInput()`
  * Returns a hidden input containing the CSRF token
* `csrfToken()`
  * Returns the current CSRF token
* `route()`
  * Returns a URL that is created using the rules of the input route name
  * Accepts the following arguments:
    1. `string $routeName` - The name of the route whose URL we're creating
    2. `array|mixed $args` - The arguments to pass into the `URLGenerator` to fill any host or path variables in the route ([learn more about the `URLGenerator`](routing#url-generators))

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

<h2 id="custom-functions">Custom Functions</h2>
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

<h2 id="using-template-functions-in-php-code">Using Template Functions in PHP Code</h2>
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