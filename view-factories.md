# View Factories

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Usage](#basic-usage)
3. [Builders](#builders)

<h2 id="introduction">Introduction</h2>
Having to always pass in the full path to load a view from a file can get annoying.  It can also make it more difficult to switch your view directory should you ever decide to do so.  This is where a `Factory` comes in handy.  

<h2 id="basic-usage">Basic Usage</h2>
Simply pass in a `FileSystem` and the directory that your views are stored in, and you'll never have to repeat yourself:
 
```php
use Opulence\Files\FileSystem;
use Opulence\Views\Factories\ViewFactory;

$fileSystem = new FileSystem();
// Assume we keep all views at "/var/www/html/views"
$factory = new ViewFactory($fileSystem, "/var/www/html/views");
// This creates a view from "/var/www/html/views/login.html"
$loginView = $factory->create("login.html");
// This creates a view from "/var/www/html/views/books/list.html"
$bookListView = $factory->create("books/list.html");
```
 
> **Note:** Preceding slashes in `create()` are not necessary.  Also, your view files can be any text extension, eg .html, .php, etc.
 
<h2 id="builders">Builders</h2>
 
Repetitive tasks such as setting up views should not be done in controllers.  That should be left to dedicated classes called `Builders`.  A `Builder` is a class that does any setup on a view after it is created by the factory.  You can register a `Builder` to a view so that each time that view is loaded by the factory, the builders are run.  Register builders via `IViewFactory::registerBuilder()`.  The second parameter is a callback that returns an instance of your builder.  Builders are lazy-loaded (ie they're only created when they're needed), which is why a callback is passed instead of the actual instance.  Your builder classes must implement `Opulence\Views\IBuilder`.  It's recommended that you register your builders via a [`Bootstrapper`](bootstrappers).

Let's take a look at an example:

```
<!-- Let's say this markup is in "Index.fortune.php" -->
<h1>{{siteName}}</h1>
{{content}}
```

```php
namespace MyApp\Builders;
use Opulence\Files\FileSystem;
use Opulence\Views\Factories\FortuneViewFactory;
use Opulence\Views\IBuilder;
use Opulence\Views\IView;

class MyBuilder implements IBuilder
{
    public function build(IView $view)
    {
        $view->setTag("siteName", "My Website");
        
        return $view;
    }
}

// Register our builder to "Index.fortune.php"
$factory = new FortuneViewFactory(new FileSystem(), __DIR__ . "/tmp");
$callback = function()
{
    return new MyBuilder();
};
$factory->registerBuilder("Index", $callback);

// Now, whenever we request "Index", the "siteName" tag will be set to "My Website"
$view = $factory->create("Index");
echo $view->getTag("siteName"); // "My Website"
```