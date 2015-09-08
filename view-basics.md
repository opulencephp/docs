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

You can set the `$user` variable in your application code using `$view->setVar("user", new User("Dave"))`.  When the view gets compiled, you'll see `Hello, Dave`.  You can also set many variables using `$view->setVars(["foo" => "bar", "baz" => "blah"])`.

<h2 id="compilers">Compilers</h2>
Compilers are what turn your raw views into valid markup, such as HTML and XML.  They must implement `Opulence\Views\Compilers\ICompiler`, which only has a single method - `compile()`.

<h4 id="registering-compilers">Registering Compilers</h4>
Opulence's `Opulence\Views\Compilers\Compiler` compiler uses an `Opulence\Views\Compilers\ICompilerRegistry` to determine which compiler to use by looking at a view's file extension.  For example, if a view file is `foo.php`, the [PHP compiler](#php-compiler) will be used.  Likewise, if a view file is named `bar.fortune`, the [Fortune compiler](#fortune-compiler) will be used.  You can register a compiler for a particular expression using `ICompilerRegistry::registerCompiler()`:

```php
use Opulence\Views\Compilers\CompilerRegistry;

$registry = new CompilerRegistry();
$registry->registerCompiler("my-extension", new MyCompiler());
```

Now, files with extension `my-extension` will be compiled by `MyCompiler`.

> **Note:** If you use the <a href="https://github.com/opulencephp/Project" target="_blank">skeleton project</a>, the PHP and Fortune compilers are already registered to the compiler registry.

<h4 id="php-compiler">PHP Compiler</h4>
If your views only use native PHP, name them with the `php` file extension, eg `MyView.php`.  Then, they'll be compiled as native PHP by Opulence.  You can still set and use variables in views when using the PHP compiler:

##### View.php
```php
<?php echo $foo; ?>
```

```php
$view->setVar("foo", "bar");
```

The PHP compiler will compile this to `bar`.

<h4 id="fortune-compiler">Fortune Compiler</h4>
Fortune is Opulence's powerful built-in view engine.  To make use Fortune, name your view file with the `fortune` file extension, eg `MyView.fortune`.  To learn more about Fortune, [read the documentation on it](view-fortune).

<h2 id="factories">Factories</h2>
Having to always pass in the full path to load a view from a file can get annoying.  It can also make it more difficult to switch your view directory should you ever decide to do so.  This is where a `Factory` comes in handy.

<h4 id="registering-resolvers">Registering Resolvers</h4>
The view factory allows you to create a view using nothing but the filename (no path or extension).  It does this using a file name resolver.  You register the path where all view files reside as well as the possible file extensions views may have.  Then, the resolver then finds the raw view file, creates a `View` object from its contents, and returns it.

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
Repetitive tasks such as setting up views should not be done in controllers.  That should be left to dedicated classes called `Builders`.  A `Builder` is a class that does any setup on a view after it is created by the factory.  You can register a `Builder` to a view so that each time that view is loaded by the factory, the builders are run.  Register builders via `IViewFactory::registerBuilder()`.  The second parameter is a callback that returns an instance of your builder.  Builders are lazy-loaded (ie they're only created when they're needed), which is why a callback is passed instead of the actual instance.  Your builder classes must implement `Opulence\Views\Factories\IViewBuilder`.  It's recommended that you register your builders via a [`Bootstrapper`](bootstrappers).

Let's take a look at an example:

##### Index.fortune

```
{{$siteName}}
```

```php
namespace MyApp\Builders;
use Opulence\Files\FileSystem;
use Opulence\Views\Factories\FortuneViewFactory;
use Opulence\Views\Factories\IViewBuilder;
use Opulence\Views\IView;

class MyBuilder implements IViewBuilder
{
    public function build(IView $view)
    {
        $view->setVar("siteName", "My Website");
        
        return $view;
    }
}

$factory->registerBuilder("Index.fortune", function()
{
    return new MyBuilder();
});

// Now, whenever we request "Index", the "siteName" variable will be set to "My Website"
$view = $factory->create("Index");
echo $view->getVar("siteName"); // "My Website"
```