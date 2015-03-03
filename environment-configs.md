# Environment Configs

## Table of Contents
1. [Introduction](#introduction)
2. [Config Structure](#config-structure)
3. [Environment Variables](#environment-variables)

<h2 id="introduction">Introduction</h2>
Sometimes, you might want to change the way your application behaves depending on whether or not it's running on a production, staging, testing, or development machine.  A common example is a database connection - each environment might have different server credentials.  By detecting the environment, you can load the appropriate credentials.  To actually detect the environment, use an `EnvironmentDetector`.  In it, you can specify rules for various environment names.  You can also detect if you're running in a console vs an HTTP connection.

<h2 id="config-structure">Config Structure</h2>
The configuration that's passed into `EnvironmentDetector::detect()` should be either:

* A callback function that returns the name of the environment the current server resides in OR
* An array that maps environment names to rules
  * Each rule must be one of the following:
    1. A server host IP or array of host IPs that belong to that environment
    2. An array containing the following keys:
      * "type" => One of the following values:
        * "regex" => Denotes that the rule uses a regular expression
      * "value" => The value of the rule, eg the regular expression to use

Let's take a look at an example:
```php
use RDev\Applications\Environments;

$configArray = [
   // Let's say that there's only one production server
   "production" => "123.456.7.8",
   // Let's say there's a list of staging servers
   "staging" => ["123.456.7.9", "123.456.7.10"],
   // Let's use a regular expression to detect a development environment
   "development" => [
       ["type" => "regex", "value" => "/^192\.168\..*$/"]
   ]
];
$detector = new Environments\EnvironmentDetector($configArray);
$environmentName = $detector->getName();
$environment = new Environments\Environment($environmentName);
```
The following is an example with a custom callback:
```php
use RDev\Applications\Environments;

$callback = function()
{
    // Look to see if a PHP environment variable was set
    if(isset($_ENV["environment"]))
    {
        return $_ENV["environment"];
    }

    // By default, return production
    return "production";
};
$detector = new Environments\EnvironmentDetector($callback);
$environmentName = $detector->getName();
$environment = new Environments\Environment($environmentName);
```

<h2 id="environment-variables">Environment Variables</h2>
Variables that are specifically tied to the environment the application is running on are called *environment variables*.  Setting an environment variable using RDev is as easy as `$environment->setVariable("foo", "bar")`.  To make configuring your environment variables as easy as possible, RDev supports environment config files, whose names are of the format ".env.DESCRIPTION_OF_CONFIG.php".  They should exist in your "configs/environment" directory.  These files are automatically run before the application is booted up.  Let's take a look at an example:
 
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