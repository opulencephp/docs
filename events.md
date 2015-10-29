# Events

## Table of Contents
1. [Introduction](#introduction)
2. [Events](#events)
3. [Listeners](#listeners)
  1. [Stopping Propagation](#stopping-propagation)
4. [Dispatchers](#dispatchers)
5. [Config](#config)

<h2 id="introduction">Introduction</h2>
There might be times in your application that you need to immediately notify certain components that an event took place.  For example, if you have "hooks" that execute before and after an action, you will need a way for components to subscribe to those hooks.  This is a great use case of events in RDev.  An event **dispatcher** holds a list of **listeners** for each type of event.  Whenever an event is fired, the dispatcher notifies all the listeners.

<h2 id="events">Events</h2>
An event object holds data about the event, which can be used by any listeners handling the event.  Events must either implement `RDev\Events\IEvent` or extend `RDev\Events\Event`.  Let's take a look at an example event to be fired when a user registers:

```php
namespace MyApp\Events;
use MyApp\Users\User;
use RDev\Events\Event;

class UserRegisteredEvent extends Event
{
    private $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }
    
    public function getUser()
    {
        return $this->user;
    }
}
```

We'll use this event in the examples below.

<h2 id="listeners">Listeners</h2>
A **listener** is what handles an event.  Listeners must be callables.  They are passed the `IEvent` object, the name of the event fired, and the event `Dispatcher`.  Let's take a look at some example listeners for our `UserRegisteredEvent`:

<h5 id="listener-closure">Listener Closure</h5>
You can use a closure for your listener:
```php
use MyApp\Events\UserRegisteredEvent;
use MyApp\Users\User;
use RDev\Events\Dispatchers\IDispatcher;

$listener = function(UserRegisteredEvent $event, $eventName, IDispatcher $dispatcher)
{
    mail($event->getUser()->getEmail(), "Welcome", "Welcome to my website!");
};
```

<h5 id="listener-class">Listener Class</h5>
You can also use a class for your listener:
```php
namespace MyApp\Events\Listeners;
use MyApp\Events\UserRegisteredEvent;
use MyApp\Users\User;
use RDev\Events\Dispatchers\IDispatcher;

class RegistrationEmail
{
    public function handle(UserRegisteredEvent $event, $eventName, IDispatcher $dispatcher)
    {
        mail($event->getUser()->getEmail(), "Welcome", "Welcome to my website!");
    }
}
```

> **Note:** Listener methods can be named whatever you'd like.

<h4 id="stopping-propagation">Stopping Propagation</h4>
There might be a case where your listener wants to prevent other listeners from being notified of an event.  In this case, your listener can call `IEvent::stopPropagation()`:

```php
$listener = function(IEvent $event)
{
    // Do stuff...
    
    $event->stopPropagation();
};
```

<h2 id="dispatchers">Dispatchers</h2>
The **event dispatcher** dispatches events to the registered listeners.  Let's add the [listener example](#listeners) to the [event example](#events):

```php
use RDev\Events\Dispatchers\Dispatcher;

$dispatcher = new Dispatcher();
$dispatcher->registerListener("user.registered", [new RegistrationEmail(), "handle"]);
```

> **Note:** Event names can be whatever you want.  In this case, we made it as descriptive as possible:  `user.registered`.

To actually fire an event, call the `dispatch()` method:

```php
use MyApp\Users\User;

$user = new User("Dave", "foo@bar.com");

// Register the user...

$dispatcher->dispatch("user.registered", new UserRegisteredEvent($user));
```

The dispatcher will loop through and call all listeners registered for the `user.registered` event.  In this case, `RegistrationEmail::handle()` will be called, and the user will receive a welcome email.

<h2 id="config">Config</h2>
If you're using the <a href="https://github.com/ramblingsofadev/Project" target="_blank">skeleton project</a>, you'll see a config array in `configs/events.php`.  The array accepts event names to an array of listeners.  A listener can be any one of the following:

* A `callable`, eg a `Closure` or an array whose first item is the listener object and whose second item is the name of the method to call in that listener
* A string with the format `Fully\Qualified\ClassName@methodName`, where the class name is the name of the listener class and the method name is the method to call in that listener