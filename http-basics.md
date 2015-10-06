# HTTP Basics

## Table of Contents
1. [Introduction](#introduction)
2. [Models](#models)
3. [Views](#views)
4. [Controllers](#controllers)
  1. [Dependency Injection](#dependency-injection)
  2. [Responses](#responses)

<h2 id="introduction">Introduction</h2>
HTTP (website) applications are the most common types of applications created with Opulence.  They make it easy to handle HTTP requests, perform business logic, render views, and send HTTP responses back to the user.  Opulence uses a powerful, yet simple way of building your HTTP applications - MVC (Model, View, and Controller).  
MVC is a programming architecture that separates your models from your views from your controllers.  This allows you to swap any component of your application without affecting the rest.  For example, you might decide to give your website a facelift.  Doing so should really only affect the views in your application.  By strictly following the MVC architecture, you can redesign your website without having to worry much at all about the backend.

<h2 id="models">Models</h2>
Models store data and the business logic behind your application.  Think of them as the heart of your application.  Your models can be plain-old PHP objects in Opulence.

<h2 id="views">Views</h2>
Views comprise the user interface portion of your application.  They should be relatively independent of your models to allow proper abstraction.  Opulence has a built-in [template engine](view-basics), although you are free to use any template engine you'd like.

<h2 id="controllers">Controllers</h2>
Controllers act as the go-between for models and views in an application.  When a model is updated, the controller updates the view.  Similarly, when a view is updated, the controller updates the models.  In Opulence, controllers can either be plain-old PHP classes, or they can extend `Opulence\Routing\Controller`, which automatically injects the HTTP request, the view factory, and the view compiler.

<h4 id="dependency-injection">Dependency Injection</h4>
Opulence uses a [dependency injection container](dependency-injection) to create controllers.  Taking advantage of this is simple:  type-hint any objects your controller needs in the controller's constructor.  Opulence will inject the appropriate objects into your controllers via your [bootstrappers](bootstrappers).

> **Note:** Primitives (eg strings and arrays) should not appear in a controller's constructor because the IoC container would have no way of resolving those dependencies at runtime.  Stick to type-hinted objects in the constructors.

Let's take a look at an example bootstrapper, controller, and view to demonstrate how controllers work:

<h5 id="bootstrapper">Bootstrapper</h5>
```php
namespace MyApp\Bootstrappers\ORM;

use MyApp\HTTP\Controllers\UserList;
use MyApp\Users\ORM\UserRepo;
use Opulence\Applications\Bootstrappers\Bootstrapper;
use Opulence\IoC\IContainer;
use Opulence\ORM\Repositories\IRepo;

class UserBootstrapper extends Bootstrapper
{
    public function registerBindings(IContainer $container)
    {
        // Bind the user repository to the UserList controller
        $container->bind(IRepo::class, UserRepo::class, UserList::class);
    }
}
```

##### Controller
```php
namespace MyApp\HTTP\Controllers;

use Opulence\HTTP\Responses\Response;
use Opulence\ORM\Repositories\IRepo;
use Opulence\Routing\Controller;

class UserList extends Controller
{
    private $users;
    
    // UserBootstrapper bound UserRepo to this controller
    // So, that's what will be injected here
    public function __construct(IRepo $users)
    {
        $this->users = $users;
    }
    
    public function showAll()
    {
        // The view factory is automatically injected by the route dispatcher
        $this->view = $this->viewFactory->create("UserList");
        $this->view->setVar("users", $this->users->getAll());
        
        // The view compiler is also automatically injected by the route dispatcher
        return new Response($this->viewCompiler->compile($this->view)); 
    }
}
```

##### UserList.fortune
```
<ul>
    <% foreach ($users as $user) %>
        <li><a href="mailto:{{ $user->getEmail() }}">{{ $user->getName() }}</a></li>
    <% endforeach %>
</ul>
```

In this example, the bootstrapper will bind `IRepo` to `UserRepo` for the `UserList` controller.  The route dispatcher will then instantiate this controller with the help of the IoC container.  The container will scan `UserList`'s constructor, realize that it needs a `UserRepo` instance, and create the `UserList` with a `UserRepo` instance.

For more information about routing, [read the documentation](routing).

<h4 id="responses">Responses</h4>
Controller methods should return an instance of the [`Opulence\HTTP\Responses\Response`](http-requests-responses) class.  If a response is not returned, whatever was returned will be wrapped up into a `200` response object.