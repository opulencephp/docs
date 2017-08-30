# Console Basics

## Table of Contents
1. [Introduction](#introduction)
2. [Running Commands](#running-commands)
3. [Getting Help](#getting-help)
4. [Built-In Commands](#built-in-commands)
  1. [app:down](#appdown)
  2. [app:env](#appenv)
  3. [app:rename](#apprename)
  4. [app:runlocally](#apprunlocally)
  5. [app:up](#appup)
  6. [composer:dump-autoload](#composerdumpautoload)
  7. [composer:update](#composerupdate)
  8. [encryption:generatepassword](#encryptiongeneratepassword)
  9. [framework:flushcache](#frameworkflushcache)
  10. [make:*](#make)
  11. [migrations:down](#migrationsdown)
  12. [migrations:up](#migrationsup)
  13. [uuid:generate](#uuidgenerate)
  14. [views:flush](#viewsflush)
  
<h2 id="introduction">Introduction</h2>
Console applications are great for administrative tasks and code generation.  Apex is Opulence's own robust console kernel.  With it, you can easily create your own console commands, display question prompts, and use HTML-like syntax for output styling.

<h2 id="running-commands">Running Commands</h2>
To run commands, type `php apex COMMAND_NAME` from the directory that Opulence is installed in.

<h2 id="getting-help">Getting Help</h2>
To get help with any command, use the help command.  There are a few different ways to call it:

```
php apex help COMMAND_NAME
php apex COMMAND_NAME -h
php apex COMMAND_NAME --help
```

<h2 id="built-in-commands">Built-In Commands</h2>
A good place to start is to run `php apex` in the directory you installed Opulence.  This will list the commands registered to the console.  Out of the box, a few commands come bundled with Apex:

<h4 id="appdown">app:down</h4>
Creates a file that lets the `Opulence\Framework\Http\Middleware\CheckMaintenanceMode` middleware know to thrown a 503 `HttpException`, which renders a "Down for maintenance" response.

<h4 id="appenv">app:env</h4>
Displays the current application environment name, eg "Production" or "Development".

<h4 id="apprename">app:rename</h4>
When you install Opulence, the default namespace is "Project".  Use this command to change this to something more fitting to your particular project.  This will update namespaces, bootstrapper names, and the composer.json PSR-4 settings.

<h4 id="apprunlocally">app:runlocally</h4>
Runs Opulence locally using the built-in PHP web server.  You can change the domain and port using the `--domain` and `--port` options, respectively.

<h4 id="appup">app:up</h4>
Deletes the file that forces the application into maintenance mode.

<h4 id="composerdumpautoload">composer:dump-autoload</h4>
This provides a wrapper around Composer's `dump-autoload` command, and is useful when another command needs to dump the autoload after running.

<h4 id="composerupdate">composer:update</h4>
A common task is updating composer and dumping the autoload.  Instead of having to run these commands manually, just run `php apex composer:update`.

<h4 id="encryptiongeneratepassword">encryption:generatepassword</h4>
The [cryptography library](cryptography#encryption) uses a password, from which cryptographic keys are derived to encrypt and decrypt data.  This command uses a <a href="https://en.wikipedia.org/wiki/Cryptographically_secure_pseudorandom_number_generator" target="_blank">CSPRNG</a> to generate a password that can be used with the encryption library.  Calling this will update the password that appears in *config/environment/.env.app.php*.  Passing a `--show` option will instead only show a new password, not update the config. 

<h4 id="frameworkflushcache">framework:flushcache</h4>
Opulence has an internal cache for several of its components:

* Bootstrappers
* Routes
* Views

If you add/remove/modify any of these components in your application, you **must** run this command to flush the internal cache.  Otherwise, your changes may not take effect.

<h4 id="make">make:*</h4>
To make creating new classes as simple as possible, Apex supports several `make:*` commands to generate class files:

1. `make:command`
2. `make:controller`
3. `make:datamapper`
4. `make:entity`
5. `make:httpmiddleware`
6. `make:migration`

They all accept a single argument: the name of the class to generate.  If you input a fully-qualified class name, then that namespace and class name will be used.  Otherwise, the default namespace will be used (eg controllers are under `Project\Http\Controllers`).

<h4 id="migrationsdown">migrations:down</h4>
This command [rolls back all executed migrations](database-migrations#rolling-back-migrations).

<h4 id="migrationsup">migrations:up</h4>
This command [runs all un-executed migrations](database-migrations#running-migrations).

<h4 id="uuidgenerate">uuid:generate</h4>
This command creates a new UUID.

<h4 id="viewsflush">views:flush</h4>
If you also use Opulence's HTTP kernel and view template, you can use this command to clear the view cache.  This is handy for when you've made updates to your views.