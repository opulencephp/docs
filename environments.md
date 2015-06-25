# Environments

## Table of Contents
1. [Introduction](#introduction)
2. [Host Registry](#host-registry)
3. [Environment Detector](#environment-detector)
4. [Environment Variables](#environment-variables)

<h2 id="introduction">Introduction</h2>
Sometimes, you might want to change the way your application behaves depending on whether or not it's running on a production, staging, testing, or development machine.  A common example is a database connection - each environment might have different server credentials.  By detecting the environment, you can load the appropriate data.

<h2 id="host-registry">Host Registry</h2>
The host registry is where you can map environment names to hosts.  The `Host` class lets you specify the host name to match against.

```php
use RDev\Applications\Environments\Hosts\Host;

$host = new Host("127.0.0.1", false);
```

You can also match against a regular expression by setting the last parameter to `true`:

```php
$host = new Host("#^127\.0\.0\.\d+$#", true);
```

Let's register the host to the registry:

```php
use RDev\Applications\Environments\Hosts\HostRegistry;

$registry = new HostRegistry();
$registry->registerHost("production", $host);
```

<h2 id="environment-detector">Environment Detector</h2>
The environment detector accepts a host registry and attempts to find the correct host in the registry.  If no matching host is found, then `"production"` is returned.  `EnvironmentDetector::detect()` returns the name of the current environment.

```php
use RDev\Applications\Environments\EnvironmentDetector;

$detector = new EnvironmentDetector();
$detector->detect($registry);
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