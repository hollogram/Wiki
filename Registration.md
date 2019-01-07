- [Overview](#overview)   
- [Usage](#usage)   
  - [The CanRegister Trait](#the-canregister-trait)   
  - [The IsVerifiable Trait](#the-isverifiable-trait)
  - [The RegisterOptions Class](#the-registeroptions-class)   
  - [The VerifyOptions Class](#the-verifyoptions-class)   
- [How To](#how-to)

# Overview

This feature provides a trait that acts as a wrapper for the `Laravel's RegistersUsers` trait.   
It's role is to give more flexibility in setting properties that are hard-coded in the framework.   
   
Please check [Laravel Authentication Documentation](https://laravel.com/docs/master/authentication) for more information about the registration process.   

# Usage

### The CanRegister Trait

Your `registration controllers` should use the `App\Traits\CanRegister` trait and implement the `getRegisterOptions()` method.   
   
The `CanRegister` trait uses as its ground-base the original `Laravel's RegistersUsers` trait.

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Traits\CanRegister;
use App\Options\RegisterOptions;

class RegisterController extends Controller
{
    use CanRegister;

    /**
     * @return RegisterOptions
     */
    public static function getRegisterOptions()
    {
        return RegisterOptions::instance()
            ->setValidator(new RegisterRequest);
    }
}
```

Please note that defining the `getRegisterOptions()` method is mandatory.   
If you fail in doing so, the code will throw an error indicating this to you.   
    
The `getRegisterOptions()` should always return an `App\Options\RegisterOptions` instance.   
You can view all options and their methods of implementation inside the `App/Options/RegisterOptions.php` file.
   
However, please remember that you are required to specify a `validator form request` by which the registration functionality will apply its validation rules.
   
You can do this using the `setValidator()` method.   

### The IsVerifiable Trait

The `App\Traits\IsVerifiable` is custom made and it hooks into the `registration process` by default.   
   
This trait is used by default on the `App\Models\Auth\User` model, enabling the `App\Traits\CanRegister` to extend its basic functionality with the new verification component.   
   
All you need to do in order to enable `email verification` when registering your application's users is to use the `App\Trait\IsVerifiable` trait on your `user models`.

```php
use App\Traits\IsVerifiable;
use App\Options\VerifyOptions;

class YourCustomUserModel
{
    use IsVerifiable;

    /**
     * Get the options for the IsVerifiable trait.
     *
     * @return VerifyOptions
     */
    public static function getVerifyOptions()
    {
        return VerifyOptions::instance();
    }
}
```

Please note that defining the `getVerifyOptions()` method is mandatory.   
If you fail in doing so, the code will throw an error indicating this to you.   
    
The `getVerifyOptions()` should always return an `App\Options\VerifyOptions` instance.   
You can view all options and their methods of implementation inside the `App/Options/VerifyOptions.php` file.

### The RegisterOptions Class

> With the help of the `getRegisterOptions()` method defined on your given controllers, you can customize the trait's behavior.   

Specify a `guard` for the registration process.

```php
/**
 * @return RegisterOptions
 */
public static function getRegisterOptions()
{
    return RegisterOptions::instance()
        ->setValidator(new RegisterRequest)
        ->setAuthGuard('admin');
}
```

Manually `disable email verification` for the registration process.   
   
By default, this is enabled. Manually disabling the email verification means that when registering a user into your application, the email verification logic will never run, regardless the fact that you used the `IsVerifiable` trait on your `user models`.

```php
/**
 * @return RegisterOptions
 */
public static function getRegisterOptions()
{
    return RegisterOptions::instance()
        ->setValidator(new RegisterRequest)
        ->disableEmailVerification();
}
```

Set the `path to redirect` after `registration`.   
   
Please note that you can also define your `after registration` logic inside the `registered(Request $request, User $user)` method from your `register controllers`. The `registered()` method takes precedence to the `registration redirect` set in your `options method`.

```php
/**
 * @return RegisterOptions
 */
public static function getRegisterOptions()
{
    return RegisterOptions::instance()
        ->setValidator(new RegisterRequest)
        ->setRegisterRedirect(route('login'));
}
```

Set the `path to redirect` after `email verification`.   
   
Please note that you can also define your `after verification` logic inside the `verified(Request $request, User $user)` method from your `register controllers`. The `verified()` method takes precedence to the `verification redirect` set in your `options method`.

```php
/**
 * @return RegisterOptions
 */
public static function getRegisterOptions()
{
    return RegisterOptions::instance()
        ->setValidator(new RegisterRequest)
        ->setVerificationRedirect(route('home'));
}
```

### The VerifyOptions Class

> With the help of the `getVerifyOptions()` method defined on your given models, you can customize the trait's behavior.   

Send your `registration verification emails` through a `queue`.   
By default, this option is no enabled.
   
First, please read about how to set queues inside the `Laravel's documentation`.

```php
/**
 * @return VerifyOptions
 */
public static function getVerifyOptions()
{
    return VerifyOptions::instance()
        ->shouldQueueEmailSending();
}
```

# How To

There is nothing custom to know here.   
   
The only thing you need to know is how the [Laravel's Authentication Process](https://laravel.com/docs/master/authentication) works.   
   
> Please note that the `verification process` is hooked automatically and requires no additional knowledge, besides the one described above.