# View Basics

## Table of Contents
1. [Introduction](#introduction)
2. [Views](#views)
  1. [Variables](#variables)
3. [Compilers](#compilers)
  1. [Registering Compilers](#registering-compilers)
  2. [PHP Compiler](#php-compiler)
  3. [Fortune Compiler](#fortune-compiler)
4. [Factories](#factories)
  1. [Registering Resolvers](#registering-resolvers)
  2. [Creating Views](#creating-views)
  3. [Builders](#builders)
  
<h2 id="introduction">Introduction</h2>
Views are the interfaces displayed to your users.  Opulence allows you to create your views using [native PHP](#php-compiler), the [Fortune compiler](#fortune-compiler) (Opulence's own view engine), or any view engine of your choice.

<h2 id="views">Views</h2>
Views implement `Opulence\Views\IView`.  They are objects that contain the uncompiled contents of the view, the path to the raw view file, and any variables set in the view.  `Opulence\Views\View` implements `IView` and comes built-in.

<h4 id="variables">Variables</h4>
Let's say you would like to output a user's name in your template:

##### View.php
```php
Hello, <?php echo $user->getName(); ?>
```

You can set the `$user` variable in your controller using

```php
$view->setVar("user", new User("Dave"));
```

When the view gets compiled, you'll see
 
```
Hello, Dave
```

You can also set many variables using 

```php
$view->setVars(["foo" => "bar", "baz" => "blah"]);
```

<h2 id="compilers">Compilers</h2>
Compilers are what turn your raw views into valid markup, such as HTML and XML.  They must implement `Opulence\Views\Compilers\ICompiler`, which only has a single method - `compile()`.

<h4 id="registering-compilers">Registering Compilers</h4>
Opulence's compiler looks at a view's file extension to determine which compiler to use.  For example, if a view file is named `foo.php`, the [PHP compiler](#php-compiler) will be used.  Likewise, if a view file is named `bar.fortune`, the [Fortune compiler](#fortune-compiler) will be used.  You can register a compiler for a particular extension using `ICompilerRegistry::registerCompiler()`:

```php
use Opulence\Views\Compilers\CompilerRegistry;

$registry = new CompilerRegistry();
$registry->registerCompiler("my-extension", new MyCompiler());
```

Now, files with extension `my-extension` will be compiled by `MyCompiler`.

> **Note:** If you use the <a href="https://github.com/opulencephp/Project" target="_blank">skeleton project</a>, the PHP and Fortune compilers are already registered to the compiler registry.

<h4 id="php-compiler">PHP Compiler</h4>
If your views only use native PHP, name them with the `php` file extension, eg `MyView.php`.  You can still set and use variables in views when using the PHP compiler:

##### View.php
```php
<?php echo $foo; ?>
```

##### Application Code
```php
$view->setVar("foo", "bar");
```

The PHP compiler will compile this to `bar`.

<h4 id="fortune-compiler">Fortune Compiler</h4>
Fortune is Opulence's powerful built-in view engine.  To use Fortune, name your view file with the `fortune` file extension, eg `MyView.fortune`.  To learn more about Fortune, [read the documentation on it](view-fortune).

<h2 id="factories">Factories</h2>
Factories simplify the way you create `View` objects from view files.

<h4 id="registering-resolvers">Registering Resolvers</h4>
The view factory allows you to create a view using nothing but the filename (no path or extension).  It does this using a file name resolver.  You register the path where all view files reside as well as the possible file extensions views may have.  Then, the resolver finds the raw view file, creates a `View` object from its contents, and returns it.

```php
use Opulence\Views\Factories\FileViewNameResolver;

$resolver = new FileViewNameResolver();
$resolver->registerPath("/var/www/html/views/some-directory");
// Register another path that has a priority, which means it will be searched first
$resolver->registerPath("/var/www/html/views/another-directory", 1);

// Register an extension
$resolver->registerExtension("php");
// Register another extension with a priority, which means it will be searched for first
$resolver->registerExtension("fortune", 1);
```
 
> **Note:** If you use the <a href="https://github.com/opulencephp/Project" target="_blank">skeleton project</a>, the `php` and `fortune` extensions are already registered to the resolver.
 
<h4 id="creating-views">Creating Views</h4>
The easiest way to create views is to pass a `ViewFactory` into the controller:

```php
use Opulence\HTTP\Responses\Response;
use Opulence\Views\Compilers\ICompiler;
use Opulence\Views\Factories\IViewFactory;

class MyController
{
    private $viewCompiler = null;
    private $viewFactory = null;
    
    public function __construct(ICompiler $compiler, IViewFactory $viewFactory)
    {
        $this->viewCompiler = $viewCompiler;
        $this->viewFactory = $viewFactory;
    }
    
    public function showHomepage()
    {
        // The view factory will search for a file named "Home" in the registered paths
        // with any of the registered extensions
        $view = $this->viewFactory->create("Home");
        $view->setVar("now", new \DateTime());
        
        return new Response($this->compiler->compile($view));
    }
}
```
 
<h4 id="builders">Builders</h4>
Your views will often need variables to be set whenever they're instantiated.  Ideally, this repetitive task should not be done in controllers.  Instead, you can use `IViewFactory::registerBuilder()` to register a view builder to be run every time a particular view is created.  The first parameter is the name of the view you're registering for.  The second parameter is a closure that accepts an `Opulence\Views\IView` object and returns a built `IView` object.

```php
$factory->registerBuilder("Homepage", function($view)
{
    $view->setVar("title", "Welcome!");
    
    return $view;
});

echo $factory->create("Homepage")->getVar("title"); // "Welcome!"
```

> **Note:** You can register as many builders as you'd like to a view.  They will be run in the order they're registered.

You can also wrap the builder into any class that you'd like:

```php
use MyApp\ProfileViewBuilder;

$factory->registerBuilder("Profile", function($view)
{
    return (new ProfileViewBuilder())->build($view);
});
```

> **Note** For convenience, Opulence provides the `Opulence\Views\Factories\IViewBuilder` interface for view builder classes.  It contains a single method `build()` where your building can take place.  However, you are not required to use `IViewBuilder`.  You can use any class you'd like.