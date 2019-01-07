- [Overview](#overview)   
- [Usage](#usage)   
  - [The CanAuthenticate Trait](#the-canauthenticate-trait)   
  - [The AuthenticateOptions Class](#the-authenticateoptions-class)   
- [How To](#how-to)

# Overview

This feature provides a trait that acts as a wrapper for the `Laravel's AuthenticatesUsers` trait.   
It's role is to give more flexibility in setting properties that are hard-coded in the framework.   
   
Please check [Laravel Authentication Documentation](https://laravel.com/docs/master/authentication) for more information about the login process.   

# Usage

### The CanAuthenticate Trait

Your `login controllers` should use the `App\Traits\CanAuthenticate` trait and implement the `getAuthenticateOptions()` method.   
   
The `CanAuthenticate` trait uses as its ground-base the original `Laravel's AuthenticatesUsers` trait.

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Traits\CanAuthenticate;
use App\Options\AuthenticateOptions;

class LoginController extends Controller
{
    use CanAuthenticate;

    /**
     * @return AuthenticateOptions
     */
    public static function getAuthenticateOptions()
    {
        return AuthenticateOptions::instance()
            ->setValidator(new LoginRequest);
    }
}
```

Please note that defining the `getAuthenticateOptions()` method is mandatory.   
If you fail in doing so, the code will throw an error indicating this to you.   
    
The `getAuthenticateOptions()` should always return an `App\Options\AuthenticateOptions` instance.   
You can view all options and their methods of implementation inside the `App/Options/AuthenticateOptions.php` file.
   
However, please remember that you are required to specify a `validator form request` by which the login functionality will apply its validation rules.
   
You can do this using the `setValidator()` method.   
   
If you fail in implementing this method, the code will throw an error indicating this to you.   

### The AuthenticateOptions Class

> With the help of the `getAuthenticateOptions()` method defined on your given models, you can customize the trait's behavior.   

Specify a `guard` for the login functionality.

```php
/**
 * @return AuthenticateOptions
 */
public static function getAuthenticateOptions()
{
    return AuthenticateOptions::instance()
        ->setValidator(new LoginRequest)
        ->setAuthGuard('admin');
}
```

Set a different `username field`.

```php
/**
 * @return AuthenticateOptions
 */
public static function getAuthenticateOptions()
{
    return AuthenticateOptions::instance()
        ->setValidator(new LoginRequest)
        ->setLogoutRedirect(route('login'));
}
```

Set the `path to redirect` after `login`.   
   
Please note that you can also define your `after login` logic inside the `authenticated(Request $request, User $user)` method from your `login controllers`. The `authenticated()` method takes precedence to the `login redirect` set in your `options method`.

```php
/**
 * @return AuthenticateOptions
 */
public static function getAuthenticateOptions()
{
    return AuthenticateOptions::instance()
        ->setValidator(new LoginRequest)
        ->setUsernameField('email`);
}
```

Set the `path to redirect` after `logout`.

```php
/**
 * @return AuthenticateOptions
 */
public static function getAuthenticateOptions()
{
    return AuthenticateOptions::instance()
        ->setValidator(new LoginRequest)
        ->setLogoutRedirect(route('login'));
}
```

Set the `maximum login attempts` before the user is timed-out.

```php
/**
 * @return AuthenticateOptions
 */
public static function getAuthenticateOptions()
{
    return AuthenticateOptions::instance()
        ->setValidator(new LoginRequest)
        ->setMaxLoginAttempts(5);
}
```

Set the `lockout time` in minutes that the user will need to wait until he can try again to login if he was timed-out.

```php
/**
 * @return AuthenticateOptions
 */
public static function getAuthenticateOptions()
{
    return AuthenticateOptions::instance()
        ->setValidator(new LoginRequest)
        ->setLockoutTime(10);
}
```

Set additional condition constraints.   
These constraints will be applied when running the login process and they will impact if the login was successful or not.

```php
/**
 * @return AuthenticateOptions
 */
public static function getAuthenticateOptions()
{
    return AuthenticateOptions::instance()
        ->setValidator(new LoginRequest)
        ->setAdditionalLoginConditions([
            'type' => User::TYPE_ADMIN,
            'verified' => User::VERIFIED_YES,
        ]);
}
```

# How To

There is nothing custom to know here.   
   
The only thing you need to know is how the [Laravel's Authentication Process](https://laravel.com/docs/master/authentication) works.