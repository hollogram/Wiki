- [Overview](#overview)   
- [Usage](#usage)   

# Overview

This feature enables you to easily define and display `flash messages` for your application.

# Usage

The `flash()` helper which actually calls the `App\Helpers\FlashHelper` class behind the scenes, is developed around `Laravel's SessionManager`, meaning it uses the `flashed sessions` to get the message and display it.   
   
Set a `success` message to be `flashed` on the next request.

```php
flash()->success('Your success message!');
```

Display the `success` message.

```php
{!! flash()->success() !!}
```
   
Set an `error` message to be `flashed` on the next request.

```php
flash()->error('Your error message!');
```

Display the `error` message.

```php
{!! flash()->error() !!}
```

Set a `warning` message to be `flashed` on the next request.

```php
flash()->warning('Your warning message!');
```

Display the `warning` message.

```php
{!! flash()->warning() !!}
```

`Display` any (`the correct`) `flash message` inside your `view file`.   

```php
{!! flash()->message() !!}

// or

{!! flash('admin')->message() !!}
```
> The `flash()` helper receives one parameter (type), which is by default instantiated to the `default` value.   
>   
> This parameter tells the `flash()` helper which view to render.   
> For the moment, only `default` and `admin` are supported as types.   
> You can find their front-end implementation inside `resources/helpers/flash/message` directory.   
>   
> Additionally you can define your own types by creating a view with the `name of the type` inside the `resources/helpers/flash/message` directory.