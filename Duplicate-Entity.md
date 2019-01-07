- [Overview](#overview)   
- [Usage](#usage)   
  - [The HasDuplicates Trait](#the-hasduplicates-trait)   
  - [The DuplicateOptions Class](#the-duplicateoptions-class)   
  - [The Eloquent Model Events](#the-eloquent-model-events)
- [How To](#how-to)   
  - [Basic Implementation](#basic-implementation)   
  - [Admin Implementation](#admin-implementation)   

# Overview

This feature provides a trait that enables you to easily replicate deeply an Eloquent model.

# Usage

### The HasDuplicates Trait

Your models should use the `App\Traits\HasDuplicates` trait and implement the `getDuplicateOptions()` method. 

```php
use App\Traits\HasDuplicates;
use App\Options\DuplicateOptions;

class YourModel
{
    use HasDuplicates;

    /**
     * Get the options for the HasDuplicates trait.
     *
     * @return DuplicateOptions
     */
    public static function getDuplicateOptions()
    {
        return DuplicateOptions::instance();
    }
}
```

Please note that defining the `getDuplicateOptions()` method is mandatory.   
If you fail in doing so, the code will throw an error indicating this to you.   
    
The `getDuplicateOptions()` should always return an `App\Options\DuplicateOptions` instance.   
You can view all options and their methods of implementation inside the `App/Options/DuplicateOptions.php` file.

### The DuplicateOptions Class

> With the help of the `getDuplicateOptions()` method defined on your given models, you can customize the trait's behavior.   

Define `excluded table columns` from being replicated on model duplication.   
This gives you the ability to `ignore certain columns` when a model is duplicated, leaving them empty.   
   
Define the excluded columns as an array, or as one or more parameters as strings.

```php
/**
 * Get the options for the HasDuplicates trait.
 *
 * @return DuplicateOptions
 */
public static function getDuplicateOptions()
{
    return DuplicateOptions::instance()
        ->excludeColumns('name', 'content');
}
```

Define `unique columns`.   
Certain table columns, such as `slug` are meant to be unique for the given model.   
   
When duplicating a model, you might want to have an unique casting on one or more columns.   
   
The defined unique columns will have a unique number appended to the original value, prefixed by a dash.   
   
Define the unique columns as an array, or as one or more parameters as strings.

```php
/**
 * Get the options for the HasDuplicates trait.
 *
 * @return DuplicateOptions
 */
public static function getDuplicateOptions()
{
    return DuplicateOptions::instance()
        ->uniqueColumns('slug', 'identifier');
}
```

Define `excluded entire model relations` that will not be replicated on model duplication.   
This gives you the ability to `ignore certain relations` from the model when that model is duplicated.   
   
By default, all defined relations for a model will be duplicated alongside with the model itself.   
A good use-case for excluding relations is with parent-type relations, or relations that need manual parsing from some reason.   
   
Define the excluded relations as an array, or as one or more parameters as strings.

```php
/**
 * Get the options for the HasDuplicates trait.
 *
 * @return DuplicateOptions
 */
public static function getDuplicateOptions()
{
    return DuplicateOptions::instance()
        ->excludeRelations('parent', 'url');
}
```

Define `unique columns for a relation` of the model.   
Certain table columns, such as `slug` are meant to be unique for the given model.   
   
When duplicating a model alongside with its relations, you might want to have an unique casting on one or more columns belonging to one or more relations to be duplicated alongside your original model.   
   
The defined unique columns will have a unique number appended to the original value, prefixed by a dash.   
   
Define the excluded relations columns as an array, where the key represents the relation name and the value represents the unique relation's columns as an array also.

```php
/**
 * Get the options for the HasDuplicates trait.
 *
 * @return DuplicateOptions
 */
public static function getDuplicateOptions()
{
    return DuplicateOptions::instance()
        ->uniqueRelationColumns([
            'products' => ['sku', 'identifier'],
            'url' => ['slug']
        ]);
}
```

Disable `deep duplication` entirely.   
   
By default, when duplicating a model all of its relations will be duplicated alongside. Disabling the deep duplication, means that only the model itself will be replicated, ignoring all of its relations.

```php
/**
 * Get the options for the HasDuplicates trait.
 *
 * @return DuplicateOptions
 */
public static function disableDeepDuplication()
{
    return DuplicateOptions::instance()
        ->uniqueRelationColumns([
            'products' => ['sku', 'identifier'],
            'url' => ['slug']
        ]);
}
```

### The Eloquent Model Events

In the duplication process of a model, `2 Eloquent events` are fired:
- `duplicating` - fired before the duplication process
- `duplicated` - fired after the duplication process
   
You can use these 2 events, just like any other Eloquent events (`saving`, `saved`, etc.) if your application logic demands it.

# How To

### Basic Implementation

After you've enabled your models to support duplication and you've customised the duplication process according to your needs, you can actually duplicate a model, using the `saveAsDuplicate()` method.   
   
The `saveAsDuplicate()` method returns the duplicated model.

```php
$model = YourModel::find($id);

$duplicatedModel = $model->saveAsDuplicate();
```

### Admin Implementation

After you've setup your models to support the duplicate feature, you'll want to enable that on your entities, inside the admin panel.   
   
You can easily accomplish that by following the above steps.   
   
In `routes/web.php` add the route responsible for managing your entity's duplication logic.

```php
Route::post('duplicate/{your_model}', ['as' => 'admin.your_entity.duplicate', 'uses' => 'YourController@duplicate', 'permissions' => 'your-entity-duplicate']);
```

In your `controller file`, add the `duplicate()` method, corresponding to the route defined above.

```php
/**
 * @param \App\Models\YourModel $model
 * @return \Illuminate\Http\RedirectResponse
 * @throws \Exception
 */
public function duplicate(YourModel $model)
{
    return $this->_duplicate(function () use ($model) {
        $this->item = $model->saveAsDuplicate();
        $this->redirect = redirect()->route('your.entity.route.for.edit');
    });
}
```

> Please note that the above example assumes you're using the `App\Traits\CanCrud` trait on your controller. Doing this makes it easier for you to implement the duplicate method.   
>   
> If you choose not to use the `App\Traits\CanCrud` trait, you can still implement this method, but it will have to contain your custom logic.

In your `_buttons.blade.php` file corresponding to the entity in question, add the `duplicate button`, enabling the admin to duplicate a model record.   

```html
<section class="actions">
    @if($item->exists)
        <!-- pass the path to your duplicate route defined above -->
        <!-- the $item represents a loaded model instance -->
        {!! button()->duplicateRecord(route('admin.your_entity.duplicate', $item->id)) !!}
    @endif

    ...
</section>
```

> Please note that the above example assumes you'll be using the `button()` helper. Doing this makes it easier for you to implement the duplicate button functionality.   
>   
> If you choose not to use the `button()` helper, you can still implement this button, but it will have to contain your custom logic.