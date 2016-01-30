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
* PHP 5.5, 5.6, 7.0, or HHVM &ge; 3.4.0
* OpenSSL
* mbstring
* A default PHP timezone set in the PHP.ini

<h2 id="installing">Installing</h2>
Opulence can be easily installed using Composer:

```php
composer create-project opulence/project --prefer-dist --stability=dev
```

Be sure to [configure your server](#server-config) to finish the installation.  Load up your website in a browser, and you should see a basic website explaining on how to start customizing it.  That's it!  If it does not show up, make sure you've made `PATH_TO_OPULENCE/tmp` writable.

> **Note:** You can <a href="https://getcomposer.org/download/" target="_blank">download Composer from here</a>.

<h4 id="libraries">Libraries</h4>
Opulence is broken into various libraries, each of which can be installed individually:

* opulence/applications
* opulence/authentication
* opulence/bootstrappers
* opulence/cache
* opulence/console
* opulence/cryptography
* opulence/databases
* opulence/debug
* opulence/environments
* opulence/events
* opulence/files
* opulence/http
* opulence/ioc
* opulence/memcached
* opulence/orm
* opulence/pipelines
* opulence/querybuilders
* opulence/redis
* opulence/routing
* opulence/sessions
* opulence/validation
* opulence/views

<h2 id="server-config">Server Config</h2>
* Opulence's `tmp` directory needs to be writable from PHP
* The document root needs to be set to Opulence's `public` directory (usually `/var/www/html/public` or `/var/www/html/YOUR_SITE_NAME/public`)

> **Note:** You must set `YOUR_SITE_DOMAIN` and `YOUR_SITE_DIRECTORY` with the appropriate values in the configs below.

<h4 id="apache-config">Apache Config</h4>
Create a virtual host in your Apache config with the following settings:

```
<VirtualHost *:80>
    ServerName YOUR_SITE_DOMAIN
    DocumentRoot YOUR_SITE_DIRECTORY/public

    <Directory YOUR_DOCUMENT_ROOT/public>
        <IfModule mod_rewrite.c>
            RewriteEngine On

            # Handle trailing slashes
            RewriteRule ^(.*)/$ /$1 [L,R=301]

            # Create pretty URLs
            RewriteCond %{REQUEST_FILENAME} !-f
            RewriteRule ^ index.php [L]
        </IfModule>
    </Directory>
</VirtualHost>
```

<h4 id="nginx-config">Nginx Config</h4>
Add the following to your Nginx config:

```
server {
    listen 80;
    server_name YOUR_SITE_DOMAIN;
    root YOUR_SITE_DIRECTORY/public;
    
    # Handle trailing slashes
    rewrite ^(.*)/$ /$1 permanent;
    
    # Create pretty URLs
    location {
        try_files $uri $uri/ /index.php?$query_string;
    }
}
```

<h2 id="renaming-project">Renaming Project</h2>
By default, an Opulence project is named "Project".  To change it to something more fitting for your application, open up a console on your server, navigate to the directory Opulence was installed to, and run:

```
php apex app:rename Project NEW_NAME
```

This will automatically update all the folders, namespaces, and Composer config to use the new name.