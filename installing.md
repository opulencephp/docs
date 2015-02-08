# Installing

## Table of Contents
1. [Requirements](#requirements)
2. [Installing](#installing)
  1. [After Installing](#after-installing)

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

> **Note:** You can [download Composer from here](https://getcomposer.org/download/).

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

<a id="after-installing"></a>
#### After Installing
After you've finished installing RDev, run `composer dump-autoload -o`.  You need to make RDev's "tmp" directory writable from PHP.  For example, run `chmod -R 777 tmp`.

If using Apache, change your Document Root to RDev's "public" directory (usually "/var/www/html/public").  Finally, enable .htaccess files in your Apache config by setting `AllowOverride` to "All" for the "public" directory.

That should be it.  Load up your website in a browser, and you should see a basic website explaining on how to start customizing it.