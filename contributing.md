# Contributing

## Table of Contents
1. [Bugs](#bugs)
  1. [Reporting a Bug](#reporting-bug)
  2. [Fixing a Bug](#fixing-bug)
2. [Features](#features)
3. [Security Vulnerabilities](#security-vulnerabilities)
4. [Coding Style](#coding-style)
  1. [PHPDoc](#phpdoc)
5. [Naming Conventions](#naming-conventions)
  1. [Variables](#variables)
  2. [Functions/Methods](#functions-methods)
  3. [Constants](#constants)
  4. [Namespaces](#namespaces)
  5. [Classes](#classes)
  6. [Abstract Classes](#abstract-classes)
  7. [Interfaces](#interfaces)
  8. [Traits](#traits)

<h2 id="bugs">Bugs</h2>
Before you attempt to write a bug fix, first read the [documentation](/docs) to see if you're perhaps using Opulence incorrectly.

<h3 id="reporting-bug">Reporting a Bug</h3>
To report a bug, <a href="https://github.com/opulencephp/Opulence/issues" target="_blank">create a new issue</a> with a descriptive title, steps to reproduce the bug (eg a failing PHPUnit test), and information about your environment.

<h3 id="fixing-bug">Fixing a Bug</h3>
To fix a bug, create a pull request on the latest stable branch with the fix and relevant PHPUnit tests.

<h2 id="features">Features</h2>
We always appreciate when you want to add a new feature to Opulence.  For minor, backwards-compatible features, create a pull request on the latest stable branch.  For major, possibly backwards-incompatible features, create a pull request on the master branch.

<h2 id="security-vulnerabilities">Security Vulnerabilities</h2>
Opulence takes security seriously.  If you find a security vulnerability, please email us at <a href="mailto:bugs@opulencephp.com">bugs@opulencephp.com</a>.

<h2 id="coding-style">Coding Style</h2>
Opulence follows PSR-1 and PSR-2 coding standards and uses PSR-4 autoloading.

<h3 id="phpdoc">PHPDoc</h3>
Use PHPDoc to document **all** class properties, methods, and functions.  Constructors only need to document the parameters.  Method/function PHPDoc must include one blank line between the description and the following tag.  Here's an example:

```php
class Book
{
    /** @var string The title of the book */
    private $title;
    
    /**
     * @param string $title The title of the book
     */
    public function __construct($title)
    {
        $this->setTitle($title);
    }
    
    /**
     * Gets the title of the book
     *
     * @return string The title of the book
     */
    public function getTitle()
    {
        return $this->title;
    }
    
    /**
     * Sets the title of the book
     *
     * @param string $title The title of the book
     * @return $this For object chaining
     */
    public function setTitle($title)
    {
        $this->title = $title;
        
        return $this;
    }
}
```

<h2 id="naming-conventions">Naming Conventions</h2>
Inspired by <a href="http://www.amazon.com/Code-Complete-Practical-Handbook-Construction/dp/0735619670" target="_blank">Code Complete</a>, Opulence uses a straightforward approach to naming things.

<h3 id="variables">Variables</h3>
All variable names:

* Must be lower camel case, eg `$emailAddress`
* Must NOT use Hungarian Notation

<h3 id="functions-methods">Functions/Methods</h3>
All function/method names:

* Must be succinct
  * Your method name should describe exactly what it does, nothing more, and nothing less
  * If you are having trouble naming a method, that's probably a sign that it is doing too much and should be refactored
* Must be lower camel case, eg `compileList()`
  * Acronyms in function/method names &le; 2 characters long, capitalize each character, eg `startIO()`
  * "Id" is an abbreviation (not an acronym) for "Identifier", so it should be capitalized `Id`
* Must answer a question if returning a boolean variable, eg `hasAccess()` or `userIsValid()`
  * Always think about how your function/method will be read aloud in an `if` statement.  `if (userIsValid())` reads better than `if (isUserValid())`.
* Must use `getXXX()` and `setXXX()` for functions/methods that get and set properties, respectively
  * Don't name a method that returns a username `username()`.  Name it `getUsername()` so that its purpose is unambiguous.

<h3 id="constants">Constants</h3>
All class constants' names:

* Must be upper snake case, eg `TYPE_SUBSCRIBER`

<h3 id="namespaces">Namespaces</h3>
All namespaces:

* Must be Pascal case, eg `Opulence\QueryBuilders`
  * For namespace acronyms &le; 2 characters long, capitalize each character, eg `IO`

<h3 id="classes">Classes</h3>
All class names:

* Must be succinct
  * Your class name should describe exactly what it does, nothing more, and nothing less
  * If you are having trouble naming a class, that's probably a sign that it is doing too much and should be refactored
* Must be Pascal case, eg `ListCompiler`
  * For class name acronyms &le; 2 characters long, capitalize each character, eg `IO`
  * Class filenames should simply be the class name with `.php` appended, eg `ListCompiler.php`
  
Class properties should appear before any methods.  The following is the preferred ordering of class properties and methods:

##### Properties
1. Constants
2. Public static properties
3. Public properties
4. Protected static properties
5. Protected properties
6. Private static properties
7. Private properties

##### Methods
1. Magic methods
2. Public static methods
3. Public abstract methods
4. Public methods
5. Protected static methods
6. Protected abstract methods
7. Protected methods
8. Private static methods
9. Private methods

> **Note:** Methods of the same visibility should be ordered alphabetically.

<h3 id="abstract-classes">Abstract Classes</h3>
All abstract class names:

* Must be Pascal case, eg `ConnectionPool`
* Must NOT use `Abstract`, `Base`, or any other word in the name that implies it is an abstract class
  
<h3 id="interfaces">Interfaces</h3>
All interface names:

* Must be preceded by an `I`, eg `IUser`

<h3 id="traits">Traits</h3>
All trait names:

* Must be preceded by a `T`, eg `TListValidator`