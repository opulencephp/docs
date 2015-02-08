# Installing

## Table of Contents
1. [Requirements](#requirements)
2. [Installing](#installing)
  1. [Libraries](#libraries)
  2. [Apache Config](#apache-config)
3. [Renaming Project](#renaming-project)

<a id="requirements"></a>
## Requirements
RDev is most easily installed using Composer:

* PHP 5.5, 5.6, or HHVM >= 3.4.0
* OpenSSL
* mbstring
* A default PHP timezone set in the PHP.ini

<a id="installing"></a>
## Installing
RDev can be easily installed using Composer:

```
composer create-project rdev/project --prefer-dist
```

Run `composer dump-autoload -o`, and then [configure Apache](#apache-config) to finish the installation.  Load up your website in a browser, and you should see a basic website explaining on how to start customizing it.  That's it!

> **Note:** You can [download Composer from here](https://getcomposer.org/download/).

<a id="libraries"></a>
#### Libraries
RDev is broken into various libraries, each of which can be installed individually:

```
rdev/applications
rdev/authentication
rdev/console
rdev/cryptography
rdev/databases
rdev/files
rdev/http
rdev/ioc
rdev/orm
rdev/sessions
rdev/users
rdev/views
```

<a id="apache-config"></a>
#### Apache Config
RDev needs a few things to be setup in Apache:

* RDev's "tmp" directory needs to be writable from PHP
  * `chmod -R 777 {PATH_TO_RDEV}/tmp` can accomplish this
* Apache's `DocumentRoot` needs to be set to RDev's "public" directory (usually "/var/www/html/public")
* .htaccess needs to be enabled by setting the Apache config's `AllowOverride` to "All" for the "public" directory

<a id="renaming-project"></a>
## Renaming Project
By default, an RDev project is named "Project".  To change it to something more fitting for your application, open up a console on your server and run:

```
php rdev app:rename Project NEW_NAME
```

This will automatically update all the folders, namespaces, and Composer config to use the new name.