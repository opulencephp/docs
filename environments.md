# Environments

## Table of Contents
1. [Introduction](#introduction)
2. [Environment Names](#environment-names)
3. [Environment Variables](#environment-variables)
4. [Environment Config Files](#environment-config-files)
  1. [Manually Set Environment Name](#manually-set-environment-name)
  2. [Automatically Set Environment Name](#automatically-set-environment-name)
      1. [Host Name](#host-name)
      2. [Host Regular Expression](#host-regular-expression)
      3. [Environment Resolver](#environment-resolver)

<h2 id="introduction">Introduction</h2>
Sometimes, you might want to change the way your application behaves depending on whether or not it's running on a production, staging, testing, or development machine.  A common example is a database connection - each environment might have different server credentials.  It is highly recommended to not check these connection strings into version control for security's sake.  Instead, you can set these values in special config files and keep them on your server outside of version control.

<h2 id="environment-names">Environment Names</h2>
Knowing which environment you're in can affect the way you run your application.  For example, you might print more debugging information to the browser if you're in a development environment.  Opulence defines four environment names out of the box, but you are free to use your own names:

* `Opulence\Environments\Environment::PRODUCTION`
* `Opulence\Environments\Environment::STAGING`
* `Opulence\Environments\Environment::TESTING`
* `Opulence\Environments\Environment::DEVELOPMENT`

To set the name, you can either pass it into the constructor:

```php
use Opulence\Environments\Environment;

$environment = new Environment(Environment::DEVELOPMENT);
```

Or, you can set it after you instantiate it:

```php
$environment->setName(Environment::DEVELOPMENT);
```

To get the environment name, call `$environment->getName()`.

<h2 id="environment-variables">Environment Variables</h2>
Variables that are specifically tied to the environment the application is running on are called **environment variables**.  Setting an environment variable using Opulence is as easy as `$environment->setVar("foo", "bar")`.  To get the environment variable, you can either call `$environment->getVar("foo")` or use PHP's native `getenv("foo")`.

<h2 id="environment-config-files">Environment Config Files</h2>
To make configuring your environment variables as easy as possible, Opulence supports environment config files, whose names are of the format ".env.DESCRIPTION_OF_CONFIG.php".  They should exist in your "config/environment" directory.  These files are automatically run before the application is booted up.  Let's take a look at an example:
 
##### .env.example.php
```php
$environment->setVar("DB_HOST", "localhost");
$environment->setVar("DB_USER", "myuser");
$environment->setVar("DB_PASSWORD", "mypassword");
$environment->setVar("DB_NAME", "public");
$environment->setVar("DB_PORT", 5432);
```

> **Note:** It is strongly recommended that production servers are setup with hard-coded environment variables in their configs.  For security, it's strongly recommended that you do not version-control your environment variable configs.  Instead, each developer should be given a template of the environment config (eg .env.example.php), and should fill out the config with the appropriate values for their environment.

When you install Opulence, you'll see two environment config files:

1. `.env.example.php`
  * Values in here will not be used - it only serves as a template
  * This can be checked into version control
2. `.env.app.php`
  * Where your actual environment variables should be stored
  * This should not be checked into version control

<h4 id="manually-set-environment-name">Manually Set Environment Name</h4>
If you're using the <a href="https://github.com/opulencephp/Project" target="_blank">skeleton project</a>, you have two ways of setting the environment name (eg "production", "development", etc):

1. Manually set it in the `ENV_NAME` environment var in your `.env.app.php`
2. [Automatically detect the environment](#automatic-environment-detection) by looking at the server's host name

Opulence defines four environment names out of the box, but you are free to use your own names:

* `Opulence\Environments\Environment::PRODUCTION`
* `Opulence\Environments\Environment::STAGING`
* `Opulence\Environments\Environment::TESTING`
* `Opulence\Environments\Environment::DEVELOPMENT`

<h4 id="automatically-set-environment-name">Automatically Set Environment Name</h4>
Opulence uses host classes to help the application determine which environment it's running in.  Hosts implement `Opulence\Environments\Hosts\IHost`.

<h5 id="host-name">Host Name</h5>
If you're adding a rule for a single, specific host, use `HostName`:

```php
use Opulence\Environments\Hosts\HostName;

$host = new HostName("127.0.0.1");
```

<h5 id="host-regular-expression">Host Regular Expression</h5>
You can use a regular expression to match hosts by using `HostRegex`:

```php
use Opulence\Environments\Hosts\HostRegex;

$host = new HostRegex("^127\.0\.0\.\d+$");
```

> **Note:** You do not need to add regular expression delimiters.  The "#" character is used as the default delimiter.

<h5 id="environment-resolver">Environment Resolver</h5>
The environment resolver registers hosts with environments and attempts to find the correct host in the registry.  If no matching host is found, then `"production"` is returned.  `EnvironmentResolver::resolve()` returns the name of the current environment.  We can use this value to set the name of the environment:

```php
use Opulence\Environments\Environment;
use Opulence\Environments\Hosts\HostName;
use Opulence\Environments\Hosts\HostRegex;
use Opulence\Environments\Resolvers\EnvironmentResolver;

$resolver = new EnvironmentResolver();
$resolver->registerHost("production", new HostName("192.168.1.1"));
$resolver->registerHost("testing", new HostRegex("^127\.0\.0\.\d+$"));
$environmentName = $resolver->resolve(gethostname());
$environment = new Environment($environmentName);
```