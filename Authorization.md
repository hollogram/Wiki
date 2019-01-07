- [Overview](#overview)   
- [Admin Entity](#admin-entity)
- [Usage](#usage)   
  - [The Permission Model](#the-permission-model)   
  - [The Role Model](#the-role-model)   
  - [The Permissions Seeder](#the-permissions-seeder)   
  - [The Roles Seeder](#the-roles-seeder)   
  - [The Permissions Middleware](#the-permissions-middleware)   
  - [The Roles Middleware](#the-roles-middleware)   
  - [The HasRoles Trait](#the-hasroles-trait)   
- [How To](#how-to)
  - [The Methods](#the-methods)   
  - [The Blade Directives](#the-blade-directives)

# Overview

This feature allows you to manage user roles and permissions in a database, for different user types (normal, admin, etc.).   
   
You will be able to:
- assign multiple roles to multiple users
- assign multiple permissions to multiple roles
- assign multiple permissions to multiple users directly

# Admin Entity

Putting aside the role based permission features that the developer will use, the `acl` component comes bundled with an `acl entity` out of the box.

You can find this section inside the `Admin -> Access Control -> Roles`, or by going to the `/admin/roles` url.

The `acl entity` allows the admin to manage all of the application's `roles` and `permissions` for the users . The features it offers are:
- create/read/update/delete roles
- assign permissions to roles
- fitler present roles by keyword, permission or date

Basically, this is a general place from where you can manage your roles and permissions throughout the application.

# Usage

### The Permission Model

User's `permissions` are managed by the `App\Models\Auth\Permission` model.   
   
Below you'll find some basic functionalities of the `App\Models\Auth\Permission`, but you can always inspect the model's functionalities yourself, or even add some new features to it.     

Create a permission.

```php
Permission::create([
    // the name of the permission
    'name' => 'create-articles',

    // the human readable name of the permission
    'label' => 'Create Articles',

    // the permission group
    // helps you group certain permissions
    // to display them afterwards, using the "getGrouped()" method.   
    'group' => 'Articles',

    // the permission type
    // helps you define permission only for a user type
    // by default, only 2 types exist: "admin" and "front"
    'type' => Permission::TYPE_ADMIN,
]);
```

Get a permission by its name.

```php
$permission = Permission::findByName('create-articles');
```

The `App\Models\Auth\Permission` model defines a `many to many` relation to the `App\Models\Auth\User` model, meaning you can fetch all users that have a certain permission.

```php
$permission = Permission::findByName('create-articles');

// get all users that have the "create-articles" permission assigned
$users = $permission->users;
```

The `App\Models\Auth\Permission` model defines a `many to many` relation to the `App\Models\Auth\Role` model, meaning you can fetch all roles that have a certain permission.

```php
$permission = Permission::findByName('create-articles');

// get all roles that have the "create-articles" permission assigned
$users = $permission->roles;
```

### The Role Model

User's `roles` are managed by the `App\Models\Auth\Role` model.   
   
Below you'll find some basic functionalities of the `App\Models\Auth\Role`, but you can always inspect the model's functionalities yourself, or even add some new features to it.     

Create a role.

```php
Role::create([
    // the name of the role
    'name' => 'content-editor',

    // the role type
    // helps you define roles only for a user type
    // by default, only 2 types exist: "admin" and "front"
    'type' => Role::TYPE_ADMIN,
]);
```

Get a role by its name.

```php
$role = Role::findByName('content-editor');
```

The `App\Models\Auth\Role` model defines a `many to many` relation to the `App\Models\Auth\User` model, meaning you can fetch all users that have a certain role.

```php
$role = Role::findByName('content-editor');

// get all users that have the "content-editor" role assigned
$users = $role->users;
```

The `App\Models\Auth\Role` model defines a `many to many` relation to the `App\Models\Auth\Permission` model, meaning you can fetch all permissions that have a certain role.

```php
$role = Role::findByName('content-editor');

// get all permissions that have the "content-editor" role assigned
$permissions = $role->permissions;
```

### The Permissions Seeder

If you want to bootstrap your application with already predefined permissions, you can do this by using the `PermissionsSeeder`.   
   
Inside the `PermissionsSeeder` there are already some `admin permissions` defined, by using the `$adminPermissions` and `$adminMap` properties.   
   
To define more `admin` permissions, just append to the `$adminMap` property, following the structure already defined.   
   
To define some `front` permissions, just add to the `$frontMap` property, following the structure already defined in the `$adminMap` property.      
   
If you have `custom user types` and you need too seed permissions for them, just `create 2 new properties` on the `PermissionsSeeder` class (`$yourTypePermissions` and `$yourTypeMap`), populate the `$yourTypeMap` property following the structure already defined in the `$adminMap` property and finally, inside the `run()` method, instantiate the `$yourTypePermissions` property to an empty collection and copy one the `foreach loops` and define your logic.

### The Roles Seeder

If you want to bootstrap your application with already predefined roles, you can do this by using the `RolesSeeder`.   
   
Inside the `RolesSeeder` there is already one `admin role` defined, by using the `$adminRoles` and `$adminMap` properties.   
   
To define more `admin` roles, just append to the `$adminMap` property, following the structure already defined.   
   
To define some `front` roles, just add to the `$frontMap` property, following the structure already defined in the `$adminMap` property.      
   
If you have `custom user types` and you need too seed roles for them, just `create 2 new properties` on the `RolesSeeder` class (`$yourTypeRoles` and `$yourTypeMap`), populate the `$yourTypeMap` property following the structure already defined in the `$adminMap` property and finally, inside the `run()` method, instantiate the `$yourTypeRoles` property to an empty collection and copy one the `foreach loops` and define your logic.

### The Permissions Middleware

Out of the box, the application exposes the `App\Http\Middleware\CheckPermissions` middleware for `checking user permissions`.   
   
Check permissions by defining the `check.permissions` middleware inside your `route group`.

```php
// the below syntax will make the permissions middleware to check if
// the logged in user has the "create-articles" and "delete-articles" permissions
// by using the "hasAllPermissions" method

Route::group([
    'middleware' => ['check.permissions:create-articles,delete-articles']
], function () {
    ...
});
```

Check permissions by defining the `permissions` array key in your `route definition`.

```php
// the below syntax will make the permissions middleware to check if
// the logged in user has the "view-contents" and "edit-articles" permissions
// by using the "hasAllPermissions" method

Route::get('/articles', [
    'as' => 'articles', 
    'uses' => 'ArticlesController@index', 
    'permissions' => 'view-articles,edit-articles'
]);
```

You can also merge the 2 methods above.

```php
// the below syntax will make the permissions middleware to check if
// the logged in user has the following permissions 
// "create-articles", "delete-articles", "view-articles" and "edit-articles"
// by using the "hasAllPermissions" method

Route::group([
    'middleware' => ['check.permissions:create-articles,delete-articles']
], function () {
    Route::get('/articles', [
        'as' => 'articles', 
        'uses' => 'ArticlesController@index', 
        'permissions' => 'view-articles,edit-articles'
    ]);
});
```

### The Roles Middleware

Out of the box, the application exposes the `App\Http\Middleware\CheckRoles` middleware for `checking user roles`.   
   
Check roles by defining the `check.roles` middleware inside your `route group`.

```php
// the below syntax will make the roles middleware to check if
// the logged in user has the "content-creator" and "content-reviewer" roles
// by using the "hasAllRoles" method

Route::group([
    'middleware' => ['check.roles:content-creator,content-reviewer']
], function () {
    ...
});
```

Check roles by defining the `roles` array key in your `route definition`.

```php
// the below syntax will make the roles middleware to check if
// the logged in user has the "article-creator" and "article-reviewer" roles
// by using the "hasAllRoles" method

Route::get('/articles', [
    'as' => 'articles', 
    'uses' => 'ArticlesController@index', 
    'roles' => 'article-creator,article-reviewer'
]);
```

You can also merge the 2 methods above.

```php
// the below syntax will make the roles middleware to check if
// the logged in user has the following roles
// "content-creator", "content-reviewer", "article-creator" and "article-reviewer"
// by using the "hasAllRoles" method

Route::group([
    'middleware' => ['check.roles:content-creator,content-reviewer']
], function () {
    Route::get('/articles', [
        'as' => 'articles', 
        'uses' => 'ArticlesController@index', 
        'roles' => 'article-creator,article-reviewer'
    ]);
});
```

### The HasRoles Trait

Your `user` models should use the `App\Traits\HasRoles` trait.

```php
use App\Traits\HasRoles;

class YourModel
{
    use HasRoles;

    ...
}
``` 

The `App\Traits\HasRoles` trait also adds a scope to your models to scope the query to certain roles

```php
// only get users with the role "content-editor"
$users = User::role('content-editor')->get();
```

# How To

### The Methods

After you've enabled your `user` models to support `role based permissions manipulation` you can do a bunch of operations.   
   
**Permission operations**

Grant permissions to a user.

```php
$user->grantPermission('create-articles');

// you can also give multiple permissions at once
$user->grantPermission('create-articles', 'edit-articles');

// you may also pass an array
$user->grantPermission(['create-articles', 'delete-articles']);
```

Grant permissions to a role.

```php
$role->grantPermission('create-articles');
```

Revoke permissions for a user.

```php
$user->revokePermission('create-articles');
```

Revoke permissions for a role.

```php
$role->revokePermission('create-articles');
```

Sync the permissions for a user.

```php
// all current permissions will be removed from the user and replaced by the array given
$user->syncPermissions(['create-articles', 'delete-articles']);
```

Sync the permissions for a role.

```php
// all current permissions will be removed from the role and replaced by the array given
$role->syncPermissions(['create-articles', 'delete-articles']);
```

Determine if a user has a certain permission.

```php
$user->hasPermission('create-articles');
```

Saved permissions will be registered with the `Illuminate\Auth\Access\Gate` class for the default guard. So you can test if a user has a permission with Laravel's default `can` function.

```php
$user->can('create-articles');
```

Determine if a user has at least one permission from a given list of permissions.

```php
$user->hasAnyPermission(Permission::all());
```

Determine if a user has all permissions from a given list of permissions.

```php
$user->hasAllPermissions(Permission::all());
```

**Role operations**

Assign a role to a user.

```php
$user->assignRoles('content-editor');

// you can also assign multiple roles at once
$user->assignRoles('content-editor', 'content-reviewer');
$user->assignRoles(['content-editor', 'content-reviewer']);
```

Remove a role from a user.

```php
$user->removeRoles('content-editor');

// you can also remove multiple roles at once
$user->removeRoles('content-editor', 'content-reviewer');
$user->removeRoles(['content-editor', 'content-reviewer']);
```

Sync the roles for a user.

```php
// all current roles will be removed from the user and replaced by the array given
$user->syncRoles(['content-editor', 'content-reviewer']);
```

Determine if a user has a certain role.

```php
$user->hasRole('content-editor');
```

Determine if a user has at least one role from a given list of roles.

```php
$user->hasAnyRole(Role::all());
```

Determine if a user has all roles from a given list of roles.

```php
$user->hasAllRoles(Role::all());
```

### The Blade Directives

This feature also adds blade directives to verify whether the currently logged in user has all or any of a given list of roles.

Check if a user has a certain permission.

```
@permission('create-articles')
    Permission granted!
@elsepermission
    Access restricted!
@endpermission

<!-- or -->

@haspermission('create-articles')
    Permission granted!
@elsehaspermission
    Access restricted!
@endhaspermission
```

Check if a user has at least one permission from a given list of permissions.

```
@hasanypermission(['create-articles', 'delete-articles])
    Permission granted!
@elsehasanypermission
    Access restricted!
@endhasanypermission
```

Check if a user has all permissions from a given list of permissions.

```
@hasallpermissions(['create-articles', 'delete-articles])
    Permission granted!
@elsehasallpermissions
    Access restricted!
@endhasallpermissions
```

Check if a user has a certain role.

```
@role('content-editor')
    Access granted!
@elserole
    Access restricted!
@endrole

<!-- or -->

@hasrole('content-editor')
    Access granted!
@elsehasrole
    Access restricted!
@endhasrole
```

Check if a user has at least one role from a given list of roles.

```
@hasanyrole(['content-editor', 'content-reviewer'])
    Access granted!
@elsehasanyrole
    Access restricted!
@endhasanyrole
```

Check if a user has all roles from a given list of roles.

```
@hasallroles(['content-editor', 'content-reviewer'])
    Access granted!
@elsehasallroles
    Access restricted!
@endhasallroles
```