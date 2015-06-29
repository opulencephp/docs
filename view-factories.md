# View Factories

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Usage](#basic-usage)
3. [Builders](#builders)
4. [Aliasing](#aliasing)

<h2 id="introduction">Introduction</h2>
Having to always pass in the full path to load a template from a file can get annoying.  It can also make it more difficult to switch your template directory should you ever decide to do so.  This is where a `Factory` comes in handy.  

<h2 id="basic-usage">Basic Usage</h2>
Simply pass in a `FileSystem` and the directory that your templates are stored in, and you'll never have to repeat yourself:
 
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
 
> **Note:** Preceding slashes in `create()` are not necessary.  Also, your template files can be any text extension, eg .html, .php, etc.
 
<h2 id="builders">Builders</h2>
 
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

<h2 id="aliasing">Aliasing</h2>
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