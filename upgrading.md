# Upgrading

## Table of Contents
1. [1.0.0-beta6](#1.0.0-beta6)
  1. [Files to Delete](#1.0.0-beta6-files-to-delete)
  2. [Files to Copy](#1.0.0-beta6-files-to-copy)
  3. [Files to Manually Update](#1.0.0-beta6-files-to-manually-update)

<h2 id="1.0.0-beta6">1.0.0-beta6</h2>
**Estimated Upgrade Time:** 20-40 minutes

This beta primarily focused on the overhauling of bootstrappers.  Bootstrappers now reside in the `Ioc` library, and no longer have dependencies on the `Opulence\Applications\Tasks\Dispatchers\ITaskDispatcher`, `Opulence\Bootstrappers\Paths`, or `Opulence\Environments\Environment` classes.

<h3 id="1.0.0-beta6-files-to-delete">Files to Delete</h3>
The following files need to be deleted manually from your server:
* tmp/framework/console/cachedBootstrapperRegistry.json
* tmp/framework/http/cachedBootstrapperRegistry.json

<h3 id="1.0.0-beta6-files-to-copy">Files to Copy</h3>
Unless you've customized any of the following files, you can just copy the updated versions from the <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6" target="_blank">skeleton project</a>.  Just be sure not to overwrite `namespace {YOUR_APP_NAME}\...;` at the top of your classes.
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/bootstrap/console/start.php" target="_blank">bootstrap/console/start.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/bootstrap/http/start.php" target="_blank">bootstrap/http/start.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/config/application.php" target="_blank">config/application.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/config/environment/.env.example.php" target="_blank">config/environment/.env.example.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/config/http/sessions.php" target="_blank">config/http/sessions.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/config/paths.php" target="_blank">config/paths.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Cache/RedisBootstrapper.php" target="_blank">src/{YOUR_APP_NAME}/Application/Bootstrappers/Cache/RedisBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Console/Commands/CommandsBootstrapper.php" target="_blank">src/{YOUR_APP_NAME}/Application/Bootstrappers/Console/Commands/CommandsBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Databases/SqlBootstrapper.php" target="_blank">src/{YOUR_APP_NAME}/Application/Bootstrappers/Databases/SqlBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Events/EventDispatcherBootstrapper.php" target="_blank">src/{YOUR_APP_NAME}/Application/Bootstrappers/Events/EventDispatcherBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Http/Routing/RouterBootstrapper.php" target="_blank">src/{YOUR_APP_NAME}/Application/Bootstrappers/Http/Routing/RouterBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Http/Sessions/SessionBootstrapper.php" target="_blank">src/{YOUR_APP_NAME}/Application/Bootstrappers/Http/Sessions/SessionBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Http/Views/ViewBootstrapper.php" target="_blank">src/{YOUR_APP_NAME}/Application/Bootstrappers/Http/Views/ViewBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Orm/UnitOfWorkBootstrapper.php" target="_blank">src/{YOUR_APP_NAME}/Application/Bootstrappers/Orm/UnitOfWorkBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Bootstrappers/Validation/ValidatorBootstrapper.php" target="_blank">src/{YOUR_APP_NAME}/Application/Bootstrappers/Validation/ValidatorBootstrapperBootstrapper.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Http/Middleware/CheckCsrfToken.php" target="_blank">src/{YOUR_APP_NAME}/Application/Http/Middleware/CheckCsrfToken.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/src/Project/Application/Http/Middleware/Session.php" target="_blank">src/{YOUR_APP_NAME}/Application/Http/Middleware/Session.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/tests/src/Project/Application/Console/IntegrationTestCase.php" target="_blank">tests/src/{YOUR_APP_NAME}/Application/Console/IntegrationTestCase.php</a>
* <a href="https://github.com/opulencephp/Project/blob/v1.0.0-beta6/tests/src/Project/Application/Http/IntegrationTestCase.php" target="_blank">tests/src/{YOUR_APP_NAME}/Application/Http/IntegrationTestCase.php</a>

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