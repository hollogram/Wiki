- [Overview](#overview)   
- [Admin Entity](#admin-entity)   
- [Usage](#usage)   
  - [The Email Model](#the-email-model)   
- [How To](#how-to)   
  - [Create New Email](#create-new-email)   

# Overview

The `emails` entity is a core component of the platform's `cms module`.   
   
This component offers a unified way and an intuitive interface for managing your emails from the `admin panel`.   
   
The emails which are created and then sent are nothing more than [Laravel Mailables](https://laravel.com/docs/5.4/mail).

# Admin Entity

Putting aside the email features that the developer will use, the email component comes bundled with a `emails entity` out of the box.   
   
You can find this section inside the `Admin -> Content Management -> Emails`, or by going to the `/admin/emails` url.   
From there, you can `add`, `edit`, `delete`, `soft-delete`, `draft` and `revision` emails.  

# Usage

### The Email Model

Emails are managed by the `App\Models\Cms\Email` model.

Below you'll find some basic functionalities of the `App\Models\Cms\Email`, but you can always inspect the model's functionalities yourself, or even add some new features to it.   

Get the default `from address` for emails.   
This first looks to see if your `company-email setting` is set and returns it. If it's not set then it just defaults to the `from.address` from the `/config/mail.php` file.

```php
Email::getFromAddress();
```

Get the default `from name` for emails.   
This first looks to see if your `company-name setting` is set and returns it. If it's not set then it just defaults to the `name` from the `/config/app.php` file.

```php
Email::getFromName();
```

Get the `from address` for a loaded email instance.

```php
$email = Email::find($id);

$email->from_address;
```

Get the `from name` for a loaded email instance.

```php
$email = Email::find($id);

$email->from_name;
```

Get the `reply to` for a loaded email instance.

```php
$email = Email::find($id);

$email->reply_to;
```

Get the `attachment` for a loaded email instance.

```php
$email = Email::find($id);

$email->attachment;
```

Get the `subject` for a loaded email instance.

```php
$email = Email::find($id);

$email->subject;
```

Get the `message` for a loaded email instance.

```php
$email = Email::find($id);

$email->message;
```

Sort emails by `newest`.

```php
$emails = Email::newest()->get();
```

Sort emails `alphabetically`.

```php
$emails = Email::alphabetically()->get();
```

Filter emails by `identifier`.

```php
$emails = Email::whereIdentifier('your-identifier')->get();
```

Filter emails by `type`.

```php
$emails = Email::whereType('your-type')->get();
```

Fetch all email `variables`.

```php
$variables = Email::getVariables();
```

Get an email by `identifier`.

```php
$email = Email::findByIdentifier('your-identifier');
```

# How To

### Create New Email

At some point in your application development process you might want to leverage mail sending. In order to do that you will need to `create an email type` (ex: `OrderShipped`).   
   
By default, the application supports `2 email types` which are `password recovery` and `email verification`.   
   
Please note that `defining an email type` doesn't mean you create an email. It means you can `create multiple emails` of this `type` from your `admin`.

Inside the `App\Models\Cms\Email` model define a new `TYPE` constant.

```php
/**
 * The constants defining the email type.
 *
 * @const
 */
 ...
 const TYPE_YOUR_TYPE = 100;
```

Inside the `App\Models\Cms\Email` model append your defined `TYPE` constant to the `$types` property.

```php
/**
 * The property defining the email types.
 *
 * @var array
 */
public static $types = [
    ...
    self::TYPE_YOUR_TYPE => 'Your Type',
];
```

Inside the `App\Models\Cms\Email` model create a `new key` for your `TYPE` inside the `$map` property.

```php
/**
 * The options available for each email type.
 *
 * --- class
 * The mailable class used for sending the email.
 *
 * --- view
 * The blade file used for rendering the email.
 * The value here will be relative to the /resources/views directory.
 *
 * --- partial
 * The blade file used in admin for rendering custom email type fields.
 * All partial files should be inside the /resources/views/admin/cms/emails/partials directory.
 *
 * --- preview_image
 * The name of the image used as email type preview in admin.
 * All of the preview images should be placed inside /resources/assets/img/admin/emails directory.
 * Running "gulp" is required for migrating new images to the /public directory.
 *
 * --- variables
 * Array of variables that the respective mail type is allowed to use.
 * Each array item defined here, should represent a key from the public static $variables array defined below.
 *
 * @var array
 */
public static $map = [
    ...

    self::TYPE_YOUR_TYPE => [
        'class' => 'App\Mail\YourEMail',
        'view' => 'emails.your_email_view',
        'preview_image' => 'your_email_preview.jpg',
        'variables' => [
            'var_1',
            'var_2',
        ],
    ],
];
```

Inside the `App\Models\Cms\Email` model create a `new variable` for each variable used in your `variables key` from your `$map` property. This can be achieved by appending `new keys` to the `$variables` property.

```php
/**
 * All the available variables to be used inside mailables as dynamic content.
 * Each of these variables may belong to more that only one mail, but the implementation may differ inside each mailable class.
 *
 * --- name
 * The visual name of the variable.
 *
 * --- label
 * Short description of what the variable represents.
 *
 * --- description
 * Longer description of what the variable represents and how it works.
 *
 * @var array
 */
public static $variables = [
    ...

    'var_1' => [
        'name' => 'Your Variable Name',
        'label' => 'Your Variable Label',
        'description' => 'Your variable description.',
    ],
    'var_2' => [
        'name' => 'Your Variable Name',
        'label' => 'Your Variable Label',
        'description' => 'Your variable description.',
    ],
];
```

After setting up your `email types` it's time to actually create the `email` itself.   
   
Inside your `app/Mail` directory create your `mailable class`.   
You can `copy the contents` of an already existing `mailable class` and paste it inside your new one, because the basic logic is 95% the same.   
> The `differences` that you should apply are:   
> - Inside the `build()` method, modify the `default subject` for the email.   
> - Inside the `parseMessage()` method, manage `your own variables` defined inside the `App\Models\Cms\Email` model for this `email type`.   
>   - Managing `your variables` basically means replacing the `[your_variable]` syntax defined in the `admin` when `creating` or `updating` an email, with `your own values`.

```php
<?php

namespace App\Mail;

use App\Models\Cms\Email;
use App\Exceptions\EmailException;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Contracts\Queue\ShouldQueue;

class PasswordRecovery extends Mailable implements ShouldQueue
{
    use Queueable, SerializesModels;

    /**
     * The email model.
     *
     * @var Email
     */
    protected $email;

    /**
     * Create a new message instance.
     *
     * @throws EmailException
     */
    public function __construct($identifier)
    {
        $this->email = Email::findByIdentifier($identifier);
    }

    /**
     * Build the message.
     *
     * @return $this
     * @throws EmailException
     */
    public function build()
    {
        $this->replyTo($this->email->reply_to);
        $this->from($this->email->from_address, $this->email->from_name);
        $this->subject($this->email->subject ?: 'Your Email Subject');

        $this->markdown($this->email->getView(), [
            'message' => $this->parseMessage(),
        ]);

        if ($this->email->attachment && uploaded($this->email->attachment)->exists()) {
            $this->attach(uploaded($this->email->attachment)->path(null, true), [
                'as' => uploaded($this->email->attachment)->load()->original_name,
            ]);
        }

        return $this;
    }

    /**
     * Parse the message for used variables.
     * Replace the variable names with the relevant content.
     *
     * @return mixed
     */
    private function parseMessage()
    {
        $message = $this->email->message;

        // parse your own variables

        return $message;
    }
}
```

Finally, inside the `resources/views` directory, you should `create` your `email view`, matching the `path` defined inside the `$map` property, more precisely inside the `view key` corresponding to your `email type`.   
   
Following the example above, you should create an email view called `your_email_view.blade.php` inside the `resources/views/emails` directory.

```html
@component('mail::message')

{!! $message !!}

@endcomponent
```