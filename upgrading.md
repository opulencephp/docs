# Upgrading

## Table of Contents
1. [1.0.0-rc4](#1.0.0-rc4)
  1. [Files to Copy](#1.0.0-rc4-files-to-copy)
  2. [Files to Manually Update](#1.0.0-rc4-files-to-manually-update)
2. [1.0.0-rc3](#1.0.0-rc3)
  1. [Files to Manually Update](#1.0.0-rc3-files-to-manually-update)
3. [1.0.0-rc1](#1.0.0-rc1)
  1. [Classes to Update](#1.0.0-rc1-classes-to-update)
4. [1.0.0-beta7](#1.0.0-beta7)
  1. [Files to Copy](#1.0.0-beta7-files-to-copy)
  2. [Files to Manually Update](#1.0.0-beta7-files-to-manually-update)
5. [1.0.0-beta6](#1.0.0-beta6)
  1. [Files to Delete](#1.0.0-beta6-files-to-delete)
  2. [Files to Copy](#1.0.0-beta6-files-to-copy)
  3. [Files to Manually Update](#1.0.0-beta6-files-to-manually-update)
  
<h2 id="1.0.0-rc3">1.0.0-rc4</h2>
**Estimated Upgrade Time:** 5 minutes

This release focused on fixing unit tests on Windows machines and making it easier to run Opulence on localhost.

<h3 id="1.0.0-rc4-files-to-copy">Files to Copy</h3>
Unless you've customized any of the following files, you can just copy the updated versions from the <a href="https://github.com/opulencephp/Project/blob/v1.0.0-rc2" target="_blank">skeleton project</a>.
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-rc3/phpunit.xml" target="_blank">phpunit.xml</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-rc3/src/Project/Application/Bootstrappers/Http/Sessions/SessionBootstrapper.php" target="_blank">src/Project/Application/Bootstrappers/Http/Sessions/SessionBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-rc3/src/Project/Application/Bootstrappers/Http/Views/ViewBootstrapper.php" target="_blank">src/Project/Application/Bootstrappers/Http/Views/ViewBootstrapper.php</a>

<h3 id="1.0.0-rc4-files-to-manually-update">Files to Manually Update</h3>

* Add `Environment::setVar("VIEW_CACHE", \Opulence\Views\Caching\FileCache::class);` to your `.env.app.php` and `.env.example.php`


<h2 id="1.0.0-rc3">1.0.0-rc3</h2>
**Estimated Upgrade Time:** 2 minutes

This release fixed an issue that would prevent users from clearing Opulence's framework cache when bootstrappers were deleted.

<h3 id="1.0.0-rc3-files-to-manually-update">Files to Manually Update</h3>
Unless you've customized any of the following files, you can just copy the updated versions from the <a href="https://github.com/opulencephp/Project/blob/v1.0.0-rc2" target="_blank">skeleton project</a>.
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-rc2/bootstrap/console/start.php" target="_blank">bootstrap/console/start.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-rc2/bootstrap/http/start.php" target="_blank">bootstrap/http/start.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-rc2/config/application.php" target="_blank">config/application.php</a>
  
<h2 id="1.0.0-rc1">1.0.0-rc1</h2>
**Estimated Upgrade Time:** 2 minutes

This release removed deprecated classes from previous betas.  If you're upgrading from [v1.0.0-beta7](#1.0.0-beta7), then follow the steps below.  If you're upgrading from older versions, first upgrade to v1.0.0-beta7.

<h3 id="1.0.0-rc1-classes-to-update">Classes to Update</h3>
The `Opulence\Events\Event` and `IEvent` classes do not exist anymore.  Instead, your event classes should be plain-old PHP objects (POPO).  So, stop extending/implementing `Event` and `IEvent`.  Any code that may have called `stopPropagation()` should also be removed.  Event listeners that were explicitly accepting an `Event` or `IEvent` instance should also be changed to accept any POPO.

<h2 id="1.0.0-beta7">1.0.0-beta7</h2>
**Estimated Upgrade Time:** 5 minutes

This beta was mostly about changing `Opulence\Environments\Environment` to a static class.  Additionally, there are some performance boosts and bug fixes.  If you're upgrading from [v1.0.0-beta6](#1.0.0-beta6), then follow the steps below.  If you're upgrading from an even older version, first upgrade to v1.0.0-beta6.

<h3 id="1.0.0-beta7-files-to-copy">Files to Copy</h3>
Unless you've customized any of the following files, you can just copy the updated versions from the <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta7" target="_blank">skeleton project</a>.
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta7/bootstrap/http/start.php" target="_blank">bootstrap/http/start.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta7/bootstrap/console/start.php" target="_blank">bootstrap/console/start.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta7/config/environment.php" target="_blank">config/environment.php</a>

<h3 id="1.0.0-beta7-files-to-manually-update">Files to Manually Update</h3>
Update any instances of `$environment->getVar(...)` to `\Opulence\Environments\Environment::getVar(...)` and `$environment->setVar(..., ...)` to  `\Opulence\Environments\Environment::setVar(..., ...)` in the following files:
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta7/config/environment/.env.app.php" target="_blank">config/environment/.env.app.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta7/config/environment/.env.example.php" target="_blank">config/environment/.env.example.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta7/config/http/sessions.php" target="_blank">config/http/sessions.php</a>

Update `$environment->getName()` to `\Opulence\Environments\Environment::getName()` in the following file:
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta7/config/http/exceptions.php" target="_blank">config/http/exceptions.php</a>

Remove `$container->bindInstance(Environment::class, $environment);` from <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta7/config/application.php" target="_blank">config/application.php</a>

Remove `$this->environment = require __DIR__ . "/../../../../../config/environment.php";` from the following files:
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta7/tests/src/Project/Application/Console/IntegrationTestCase.php" target="_blank">tests/src/{PROJECT_NAME}/Application/Console/IntegrationTestCase.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta7/tests/src/Project/Application/Http/IntegrationTestCase.php" target="_blank">tests/src/{PROJECT_NAME}/Application/Http/IntegrationTestCase.php</a>

<h2 id="1.0.0-beta6">1.0.0-beta6</h2>
**Estimated Upgrade Time:** 20-40 minutes

This beta primarily focused on the overhauling of bootstrappers.  Bootstrappers now reside in the `Ioc` library, and no longer have dependencies on the `Opulence\Applications\Tasks\Dispatchers\ITaskDispatcher`, `Opulence\Bootstrappers\Paths`, or `Opulence\Environments\Environment` classes.

<h3 id="1.0.0-beta6-files-to-delete">Files to Delete</h3>
The following files need to be deleted manually from your server:
* tmp/framework/console/cachedBootstrapperRegistry.json
* tmp/framework/http/cachedBootstrapperRegistry.json

<h3 id="1.0.0-beta6-files-to-copy">Files to Copy</h3>
Unless you've customized any of the following files, you can just copy the updated versions from the <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6" target="_blank">skeleton project</a>.  Just be sure not to overwrite `namespace {PROJECT_NAME}\...;` at the top of your classes.
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/bootstrap/console/start.php" target="_blank">bootstrap/console/start.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/bootstrap/http/start.php" target="_blank">bootstrap/http/start.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/config/application.php" target="_blank">config/application.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/config/environment/.env.example.php" target="_blank">config/environment/.env.example.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/config/http/sessions.php" target="_blank">config/http/sessions.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/config/paths.php" target="_blank">config/paths.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Cache/RedisBootstrapper.php" target="_blank">src/{PROJECT_NAME}/Application/Bootstrappers/Cache/RedisBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Console/Commands/CommandsBootstrapper.php" target="_blank">src/{PROJECT_NAME}/Application/Bootstrappers/Console/Commands/CommandsBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Databases/SqlBootstrapper.php" target="_blank">src/{PROJECT_NAME}/Application/Bootstrappers/Databases/SqlBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Events/EventDispatcherBootstrapper.php" target="_blank">src/{PROJECT_NAME}/Application/Bootstrappers/Events/EventDispatcherBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Http/Routing/RouterBootstrapper.php" target="_blank">src/{PROJECT_NAME}/Application/Bootstrappers/Http/Routing/RouterBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Http/Sessions/SessionBootstrapper.php" target="_blank">src/{PROJECT_NAME}/Application/Bootstrappers/Http/Sessions/SessionBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Http/Views/ViewBootstrapper.php" target="_blank">src/{PROJECT_NAME}/Application/Bootstrappers/Http/Views/ViewBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Orm/UnitOfWorkBootstrapper.php" target="_blank">src/{PROJECT_NAME}/Application/Bootstrappers/Orm/UnitOfWorkBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Validation/ValidatorBootstrapper.php" target="_blank">src/{PROJECT_NAME}/Application/Bootstrappers/Validation/ValidatorBootstrapperBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Http/Middleware/CheckCsrfToken.php" target="_blank">src/{PROJECT_NAME}/Application/Http/Middleware/CheckCsrfToken.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Http/Middleware/Session.php" target="_blank">src/{PROJECT_NAME}/Application/Http/Middleware/Session.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/tests/src/Project/Application/Console/IntegrationTestCase.php" target="_blank">tests/src/{PROJECT_NAME}/Application/Console/IntegrationTestCase.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/tests/src/Project/Application/Http/IntegrationTestCase.php" target="_blank">tests/src/{PROJECT_NAME}/Application/Http/IntegrationTestCase.php</a>

<h3 id="1.0.0-beta6-files-to-manually-update">Files to Manually Update</h3>
* In your application's bootstrappers, change:
  * `use Opulence\Bootstrappers\Bootstrapper;` to `use Opulence\Ioc\Bootstrappers\Bootstrapper;`
  * `use Opulence\Bootstrappers\ILazyBootstrapper;` to `use Opulence\Ioc\Bootstrappers\ILazyBootstrapper;`
  * `$this->environment->getVar({VAR_NAME})` to `getenv({VAR_NAME})`
  * `$this->environment->getName()` to `getenv("ENV_NAME")`
  * `$this->paths[{PATH}]` to `Config::get("paths", {PATH})`
* In config/environment/.env.app.php, add:
  * `$environment->setVar("SESSION_COOKIE_DOMAIN", "");`
  * `$environment->setVar("SESSION_COOKIE_IS_SECURE", false);`
  * `$environment->setVar("SESSION_COOKIE_PATH", "/");`