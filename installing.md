# Installing

## Table of Contents
1. [Requirements](#requirements)
2. [Installing](#installing)
  1. [Libraries](#libraries)
3. [Server Config](#server-config)
  1. [PHP Built-in Web Server Config](#php-built-in-web-server-config)
  2. [Apache Config](#apache-config)
  3. [Nginx Config](#nginx-config)
  4. [Caddy Config](#caddy-config)
4. [Routes](#routes)
5. [Views](#views)
6. [Renaming Project](#renaming-project)
7. [Console Commands](#console-commands)
8. [Environment Variables](#environment-variables)
9. [Versioning](#versioning)

<h2 id="requirements">Requirements</h2>

* PHP &ge; 7.3.0
* OpenSSL
* mbstring

> **Note:** If you're a FreeBSD user, you'll also need to make sure the ctype, dom, filter, hash, json, phar, session, tokenizer, and xml extensions are installed.

<h2 id="installing">Installing</h2>

Opulence can be easily installed using Composer:

```php
composer create-project opulence/project --prefer-dist
```

Be sure to [configure your server](#server-config) to finish the installation.  Load up your website in a browser, and you should see a basic website explaining on how to start customizing it.  That's it!  If it does not show up, make sure you've made *PATH_TO_OPULENCE/tmp* writable.

> **Note:** You can <a href="https://getcomposer.org/download/" target="_blank">download Composer from here</a>.

<h4 id="libraries">Libraries</h4>

Opulence is broken into various libraries, each of which can be installed individually:

* opulence/authentication
* opulence/authorization
* opulence/cache
* opulence/collections
* opulence/console
* opulence/cryptography
* opulence/databases
* opulence/debug
* opulence/environments
* opulence/events
* opulence/http
* opulence/io
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

* Opulence's _tmp_ directory needs to be writable from PHP
* The document root needs to be set to Opulence's _public_ directory (usually _/var/www/html/public_ or */var/www/html/YOUR_SITE_NAME/public*)

> **Note:** You must set `YOUR_SITE_DOMAIN` and `YOUR_SITE_DIRECTORY` with the appropriate values in the configs below.

<h4 id="php-built-in-web-server-config">PHP Built-in Web Server Config</h4>

To run Opulence locally, use the following command:

```
php apex app:runlocally
```
    
This will run PHP's built-in web server. The site will be accessible at http://localhost.

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
    index index.php;
    
    # Handle trailing slashes
    rewrite ^/(.*)/$ /$1 permanent;
    
    # Create pretty URLs
    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }
    
    location ~ \.php$ {
        include                 /etc/nginx/fastcgi_params;
        fastcgi_index           index.php;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_param           SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass            unix:/run/php/php7.0-fpm.sock;
    }
}
```

<h4 id="caddy-config">Caddy Config</h4>

Add the following to your Caddyfile config:

```
YOUR_SITE_DOMAIN:80 {
    rewrite {
        r .*
        ext /
        to /index.php?{query}
    }
    fastcgi / 127.0.0.1:9000 php {
        ext .php
        index index.php
    }
}
```

<h2 id="routes">Routes</h2>

Create new routes in _config/http/routes.php_. To handle each route, add methods to _src/Application/Http/Controllers/Tutorial.php_.

<h2 id="views">Views</h2>

To change the contents of each page, change the views in the _resources/views_ directory. To change the CSS, edit _public/assets/css/style.css_.

If you want to create view builders, add them to _src/Application/Http/Views/Builders_. Then, register each view builder to the appropriate template in _src/Application/Bootstrappers/Http/Views/BuildersBootstrapper.php_.

<h2 id="console-commands">Console Commands</h2>

You can run console commands running `php apex` from your project's root directory. To create a custom command, create a class that extends `Opulence\Console\Commands\Command`, and put it in the _src/Application/Console/Commands_ directory. Then, add the fully-qualified name of your command class to _config/console/commands.php_.

<h2 id="environment-variables">Environment Variables</h2>

Your application is currently in the development environment. To change the environment to production, update _config/environment/.env.app.php_ and change the `ENV_NAME` value to `Environment::PRODUCTION`.

<h2 id="renaming-project">Renaming Project</h2>

By default, an Opulence project is named "Project".  To change it to something more fitting for your application, open up a console on your server, navigate to the directory Opulence was installed to, and run:

```
php apex app:rename Project NEW_NAME
```

This will automatically update all the folders, namespaces, and Composer config to use the new name.

<h2 id="versioning">Versioning</h2>

Opulence follows semantic versioning 2.0.0.  For more information on semantic versioning, check out its <a href="http://semver.org/" title="Semantic versioning documentation" target="_blank">documentation</a>.
