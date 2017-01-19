# Environments

## Table of Contents
1. [Introduction](#introduction)
2. [Environment Names](#environment-names)
3. [Environment Variables](#environment-variables)
4. [Environment Config Files](#environment-config-files)

<h2 id="introduction">Introduction</h2>
Sometimes, you might want to change the way your application behaves depending on whether or not it's running on a production, staging, testing, or development machine.  A common example is a database connection - each environment might have different server credentials.  It is highly recommended to not check these connection strings into version control for security's sake.  Instead, you can set these values in special config files and keep them on your server outside of version control.

<h2 id="environment-names">Environment Names</h2>
Knowing which environment you're in can affect the way you run your application.  For example, you might print more debugging information to the browser if you're in a development environment.  Opulence defines four environment names out of the box, but you are free to use your own names:

* `Opulence\Environments\Environment::PRODUCTION`
* `Opulence\Environments\Environment::STAGING`
* `Opulence\Environments\Environment::TESTING`
* `Opulence\Environments\Environment::DEVELOPMENT`

If you're using the <a href="https://github.com/opulencephp/Project" target="_blank">skeleton project</a>, the environment name is stored in the `ENV_NAME` environment variable.  To access the name, call `Environment::getVar("ENV_NAME")`.

<h2 id="environment-variables">Environment Variables</h2>
Variables that are specifically tied to the environment the application is running on are called **environment variables**.  Setting an environment variable using Opulence is as easy as `Environment::setVar("foo", "bar")`.  To get the environment variable, you can either call `Environment::getVar("foo")` or use PHP's native `getenv("foo")`.  `getVar()` lets you specify a default value if one did not exist, eg `Environment::getVar("foo", "bar")`.

> **Note:** `Environment::setVar()` does not overwrite previously-defined environment values.

<h2 id="environment-config-files">Environment Config Files</h2>
To make configuring your environment variables as easy as possible, Opulence supports environment config files, whose names are of the format ".env.DESCRIPTION_OF_CONFIG.php".  They should exist in your "config/environment" directory.  These files are automatically run before the application is booted up.  Let's take a look at an example:

##### .env.example.php
```php
Environment::setVar('DB_HOST', 'localhost');
Environment::setVar('DB_USER', 'myuser');
Environment::setVar('DB_PASSWORD', 'mypassword');
Environment::setVar('DB_NAME', 'public');
Environment::setVar('DB_PORT', 5432);
```

> **Note:** It is strongly recommended that production servers are setup with hard-coded environment variables in their configs.  For security, it's strongly recommended that you do not version-control your environment variable configs.  Instead, each developer should be given a template of the environment config (eg .env.example.php), and should fill out the config with the appropriate values for their environment.

When you install Opulence, you'll see two environment config files:

1. *.env.example.php*
  * Values in here will not be used - it only serves as a template
  * This can be checked into version control
2. *.env.app.php*
  * Where your actual environment variables should be stored
  * This should not be checked into version control
