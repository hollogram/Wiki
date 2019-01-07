- [Overview](#overview)   
- [Configuration](#configuration)
- [Admin Entity](#admin-entity)
- [Usage](#usage)   
  - [The Activity Model](#the-activity-model)
  - [The HasActivity Trait](#the-hasactivity-trait)   
  - [The ActivityOptions Class](#the-activityoptions-class)    
  - [The ActivityLog Helper](#the-activitylog-helper)
- [How To](#how-to)

# Overview

This feature provides a trait that will automatically generate `activity logs` for various Eloquent models, using the Eloquent model events defined for "after save" operations.

# Configuration

You can `enable` or `disable` the entire activity log functionality for the application, by setting the `ENABLE_ACTIVITY_LOG` configuration option to `true` or `false` from inside your `.env` file.
   
You can configure your activity behavior from inside the `/config/activity.php` file.   
All options are well documented, so it should not be a problem figuring out what they mean.   

# Admin Entity

Putting aside the activity log features that the developer will use, the activity component comes bundled with an `activity entity` out of the box.   
   
You can find this section inside the `Admin -> Access Control -> Activity`, or by going to the `/admin/activity` url.

The activity entity allows the admin to manage all of the application's logged activities.   
   
The features it offers are:   
- viewing the application's logged activities   
- filter present activity logs by keyword, user and date   
- cleanup activity logs older than your specified value inside the config file   
- remove all activity logs at once

Basically, this is a general place from where you can manage your activity logs throughout the application.

# Usage

### The Activity Model

Activity logs are managed by the `App\Models\Auth\Activity` model.

Below you'll find some basic functionalities of the `App\Models\Auth\Activity`, but you can always inspect the model's functionalities yourself, or even add some new features to it.

Get the `user that caused` the activity.   
The `Activity` model defines a `one to one` relation to the `User` model.

```php
$activity->causer; // returns a "App\Models\Auth\User" model
```

Get the `subjected entity record` of an activity.   
The `Activity` model defines a `morphed to` relation, while the `HasActivity` trait defines a `morphed many` relation to the `Activity` model itself.

```php
$activity->subject; // returns the model instance the activity was created for
```

Filter activities by `causer`.

```php
Activity::causedBy(User::find($id))->get();
```

Filter activities by `subject`.

```php
Activity::forSubject(YourModel::find($id))->get();
```

### The HasActivity Trait

Your models should use the `App\Traits\HasActivity` trait and implement the `getActivityOptions()` method.   

```php
use App\Traits\HasActivity;
use App\Options\ActivityOptions;

class YourModel
{
    use HasActivity;

    /**
     * Get the options for the HasActivity trait.
     *
     * @return ActivityOptions
     */
    public static function getActivityOptions()
    {
        return ActivityOptions::instance();
    }
}
```

Please note that defining the getActivityOptions()` method is mandatory.   
If you fail in doing so, the code will throw an error indicating this to you.   
    
The `getActivityOptions()` should always return an `App\Options\ActivityOptions` instance.   
You can view all options and their methods of implementation inside the `App/Options/ActivityOptions.php` file.   

### The ActivityOptions Class

> With the help of the `getActivityOptions()` method defined on your given models, you can customize the trait's behavior.   
   
Only `log certain event actions` for a model.   
   
By default, all Eloquent events will be logged:
- `created`, `updated`, `deleted` - always
- `restored`, `drafted`, `revisioned`, `duplicated` - if functionality exists

```php
/**
 * Get the options for the HasActivity trait.
 *
 * @return ActivityOptions
 */
public static function getActivityOptions()
{
    return ActivityOptions::instance()
        ->logOnlyForTheseEvents('created', 'updated');
}
```

### The ActivityLog Helper

You can use the `activity_log` helper to quickly generate activity logs inside the `activity database table`.

```php
activity_log()
    ->causedBy(User::find($id))
    ->performedOn(YourModel::find($id))
    ->log('Created a new activity log');
```

# How To

Get all activity logs for a model instance.

```php
$model = YourModel::find($id);

// return all activity logs for the loaded model
// using the "morph many activity" relation
$model->activity;
```

Customize the model's log name.   
   
The `HasActivity` trait defines the `getLogName()` method.   
This method is responsible for building the text to be inserted inside the `name column` of the `activity database table`, when various saving Eloquent model events are triggered.   
   
You can customize the `log name` by implementing the `getLogName()` method directly on your models, overriding its defaults.

```php
/**
 * Compose the log name.
 *
 * @param string|null $event
 * @return string
 */
public function getLogName($event = null)
{
    return 'my custom log name for the ' . $event . ' event';
}
```