# Installing

## Table of Contents
1. [Requirements](#requirements)
2. [Installing](#installing)
  1. [Libraries](#libraries)
3. [Server Config](#server-config)
  1. [Apache Config](#apache-config)
  2. [Nginx Config](#nginx-config)
4. [Renaming Project](#renaming-project)

<h2 id="requirements">Requirements</h2>
* PHP 5.5, 5.6, 7.0, or HHVM >= 3.4.0
* OpenSSL
* mbstring
* A default PHP timezone set in the PHP.ini

<h2 id="installing">Installing</h2>
RDev can be easily installed using Composer:

```
composer create-project rdev/project --prefer-dist
```

Be sure to [configure Apache](#apache-config) to finish the installation.  Load up your website in a browser, and you should see a basic website explaining on how to start customizing it.  That's it!  If it does not show up, make sure you've made `{PATH_TO_RDEV}/tmp` writable.

> **Note:** You can <a href="https://getcomposer.org/download/" target="_blank">download Composer from here</a>.

<h4 id="libraries">Libraries</h4>
RDev is broken into various libraries, each of which can be installed individually:

```
rdev/applications
rdev/authentication
rdev/console
rdev/cryptography
rdev/databases
rdev/files
rdev/forms
rdev/http
rdev/ioc
rdev/memcached
rdev/orm
rdev/pipelines
rdev/querybuilders
rdev/redis
rdev/routing
rdev/sessions
rdev/users
rdev/views
```

<h2 id="server-config">Server Config</h2>
* RDev's `tmp` directory needs to be writable from PHP
* The document root needs to be set to RDev's `public` directory (usually `/var/www/html/public` or `/var/www/html/YOUR_SITE_NAME/public`)

<h4 id="apache-config">Apache Config</h4>
Create a virtual host in your Apache config with the following settings:

```
<VirtualHost *:80>
    ServerName YOUR_SITE_DOMAIN
    DocumentRoot YOUR_SITE_DIRECTORY/public
    
    # Create pretty URLs
    RewriteEngine On
    
    # Don't remove trailing slashes for API docs
    RewriteRule ^api/ - [L]

    # Handle trailing slashes
    RewriteRule ^(.*)/$ /$1 [L,R=301]

    # Boot up the application
    RewriteRule ^index.php - [L]
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^(.*)$ index.php [QSA,L]
</VirtualHost>
```

<h4 id="nginx-config">Nginx Config</h4>
Add the following to your Nginx config:

```
server {
    server_name YOUR_SITE_DOMAIN
    root YOUR_SITE_DIRECTORY/public
    
    # Create pretty URLs
    location {
        try_files $uri $uri/ /index.php?$query_string;
    }
}
```

<h2 id="renaming-project">Renaming Project</h2>
By default, an RDev project is named "Project".  To change it to something more fitting for your application, open up a console on your server and run:

```
php rdev app:rename Project NEW_NAME
```

This will automatically update all the folders, namespaces, and Composer config to use the new name.