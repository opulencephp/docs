# Authorization

## Table of Contents
1. [Introduction](#introduction)
2. [Roles](#roles)
  1. [Creating Roles](#creating-roles)
  2. [Deleting Roles](#deleting-roles)
  3. [Assigning Roles](#assigning-roles)
  4. [Revoking Roles](#revoking-roles)
  5. [Getting Subjects by Role](#getting-subjects-by-role)
3. [Permissions](#permissions)
  1. [Registering a Permission with a Role](#registering-permission-with-role)
  2. [Registering a Callback](#registering-a-callback)
  3. [Registering an Overriding Callback](#registering-overriding-callback)
4. [Authority](#authority)
  1. [Checking Permissions](#checking-permissions)

<h2 id="introduction">Introduction</h2>

The authorization library simplifies how you check if a user should have access to resources.  It provides many models that are common to most authorization schemes.  It is meant to be used in conjunction with standards like OAuth2.  With community help, it is possible for full OAuth2 support to eventually be provided out of the box.

<h2 id="roles">Roles</h2>

A role is a way of describing what a subject is.  Usually, roles, such as "editor" or "administrator" also have permissions.  Rather than having to assign a permission to each subject, you can group those users by roles, and then assign a permission to that role.

To use Opulence's authorization library, you must implement:

* `Opulence\Authorization\Roles\Orm\IRoleMembershipRepository`
  * Stores and retrieves subjects' memberships to certain roles
* `Opulence\Authorization\Roles\Orm\IRoleRepository`
  * Stores and retrieves roles

The heart of roles is `Opulence\Authorization\Roles\Roles`.  It can create roles, delete roles, assign roles to subjects, and retrieve subjects by role.  It simply requires the two repositories above.

```php
use Opulence\Authorization\Roles\Orm\IRoleMembershipRepository;
use Opulence\Authorization\Roles\Orm\IRoleRepository;
use Opulence\Authorization\Roles\Roles;

// Assume $roleMembershipRepository and $roleRepository were previously set
$roles = new Roles($roleMembershipRepository, $roleRepository);
```

<h4 id="creating-roles">Creating Roles</h4>

```php
$roles->createRole('new-role'); // Returns the new Role object
```

<h4 id="deleting-roles">Deleting Roles</h4>

```php
$roles->deleteRole('role-to-delete');
```

<h4 id="assigning-roles">Assigning Roles</h4>

```php
// The Id of the subject that is getting this role
$subjectId = 123;
// You can also pass in an array of role names to assign
$roles->assignRoles($subjectId, 'new-role');
```

<h4 id="revoking-roles">Revoking Roles</h4>

You can revoke a single role from a subject:
```php
// The Id of the subject whose role is being revoked
$subjectId = 123;
// You can also pass in an array of role names to revoke
$roles->removeRolesFromSubject($subjectId, 'revoked-role');
```

You can also revoke all roles from a subject:
```php
// The Id of the subject whose roles are being revoked
$subjectId = 123;
$roles->removeAllRolesFromSubject($subjectId);
```

<h4 id="getting-subjects-by-role">Getting Subjects by Role</h4>

```php
$subjectIdsWithRole = $roles->getSubjectIdsWithRole('search-role');
```

<h2 id="permissions">Permissions</h2>

Permissions are privileges granted to roles.  Opulence provides `Opulence\Authorization\Permissions\PermissionRegistry` as a class to store the mapping of roles to permissions.

```php
use Opulence\Authorization\Permissions\PermissionRegistry;

$registry = new PermissionRegistry();
```

<h4 id="registering-permission-with-role">Registering a Permission with a Role</h4>

```php
$registry->registerRoles('create-posts', ['editor', 'author']);
```

<h4 id="registering-a-callback">Registering a Callback</h4>

If your permission logic is a bit more involved, you can register a callback to evaluate if a subject has a permission:

```php
// Must accept the subject Id as a parameter
// Must return true if the role has the permission, otherwise false
// Assume $userRepository is some user repository
$callback = function ($subjectId) use ($userRepository) {
    $user = $userRepository->getById($subjectId);

    return $user->isACoolGuy();
};
$registry->registerCallback('create-users', $callback);
```

<h4 id="registering-overriding-callback">Registering an Overriding Callback</h4>

If you would like to short-circuit the permission checks, you may register what's called an "overriding callback".  This is most useful in the case of admins who have access to all permissions.

```php
$callback = function ($subjectId, string $permission) {
    // Let's say that subject with Id 1 is the admin
    return $subjectId === 1;
};
$registry->registerOverridingCallback($callback);
```

<h2 id="authority">Authority</h2>

`Opulence\Authorization\Authority` provides the ability to check if a user has certain permissions.

```php
use Opulence\Authorization\Authority;
use Opulence\Authorization\Permissions\PermissionRegistry;
use Opulence\Authorization\Roles\Roles;

$permissionRegistry = new PermissionRegistry();

// Set up the permission registry...

$authority = new Authority(
    $subjectId,
    $roles->getRolesForSubject($subjectId),
    $permissionRegistry
);
```

<h4 id="checking-permissions">Checking Permissions</h4>

```php
if ($authority->can('create-posts')) {
    $postRepository->save();
}
```

You can also check if a subject does not have a permission:

```php
if ($authority->cannot('create-posts')) {
    die('You do not have permission to create posts');
}
```

If you'd like to check permissions on another subject, you may do so:

```php
if ($authority->forSubject($subjectId, $subjectRoles)->can('delete-posts')) {
    $postRepository->save();
}
```
