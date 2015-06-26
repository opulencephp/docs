# Environments

## Table of Contents
1. [Introduction](#introduction)
2. [Hosts](#hosts)
  1. [Host Name](#host-name)
  2. [Host Regular Expression](#host-regular-expression)
3. [Environment Detector](#environment-detector)
4. [Environment Variables](#environment-variables)

<h2 id="introduction">Introduction</h2>
Sometimes, you might want to change the way your application behaves depending on whether or not it's running on a production, staging, testing, or development machine.  A common example is a database connection - each environment might have different server credentials.  By detecting the environment, you can load the appropriate data.

<h2 id="hosts">Hosts</h2>
RDev uses host classes to help the application determine which environment it's running in.  Hosts implement `RDev\Applications\Environments\Hosts\IHost`.

<h3 id="host-name">Host Name</h3>
If you're adding a rule for a single, specific host, use `HostName`:

```php
use RDev\Applications\Environments\Hosts\HostName;

$host = new HostName("127.0.0.1");
```

<h3 id="host-regular-expression">Host Regular Expression</h3>
You can use a regular expression to match hosts by using `HostRegex`:

```php
use RDev\Applications\Environments\Hosts\HostRegex;

$host = new HostRegex("^127\.0\.0\.\d+$");
```

> **Note:** You do not need to add regular expression delimiters.  The "#" character is used as the default delimiter.

<h2 id="environment-detector">Environment Detector</h2>
The environment detector registers hosts with environments and attempts to find the correct host in the registry.  If no matching host is found, then `"production"` is returned.  `EnvironmentDetector::detect()` returns the name of the current environment.

```php
use RDev\Applications\Environments\EnvironmentDetector;
use RDev\Applications\Environments\Hosts\HostName;
use RDev\Applications\Environments\Hosts\HostRegex;

$detector = new EnvironmentDetector();
$detector->registerHost("production", new HostName("192.168.1.1"));
$detector->registerHost("testing", new HostRegex(^"127\.0\.0\.\d+$"));
$detector->detect();
```

You can use the result of `detect()` to set the environment name via `RDev\Applications\Environments\Environment::setName()`.

<h2 id="environment-variables">Environment Variables</h2>
Variables that are specifically tied to the environment the application is running on are called **environment variables**.  Setting an environment variable using RDev is as easy as `$environment->setVariable("foo", "bar")`.  To make configuring your environment variables as easy as possible, RDev supports environment config files, whose names are of the format ".env.DESCRIPTION_OF_CONFIG.php".  They should exist in your "configs/environment" directory.  These files are automatically run before the application is booted up.  Let's take a look at an example:
 
##### .env.example.php
```php
$environment->setVariable("DB_HOST", "localhost");
$environment->setVariable("DB_USER", "myuser");
$environment->setVariable("DB_PASSWORD", "mypassword");
$environment->setVariable("DB_NAME", "public");
$environment->setVariable("DB_PORT", 5432);
```

> **Note:** For performance reasons, .env.*.php files are only loaded on non-production servers.  It is strongly recommended that production servers are setup with hard-coded environment variables in their configs.  For security, it's strongly recommended that you do not version-control your environment variable configs.  Instead, each developer should be given a template of the environment config (eg .env.example.php), and should fill out the config with the appropriate values for their environment.

When you install RDev, you'll see two environment config files:

1. `.env.example.php`
  * Values in here will not be used - it only serves as a template
  * This can be checked into version control
2. `.env.app.php`
  * Where your actual environment variables should be stored
  * This should not be checked into version control