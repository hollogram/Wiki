- [Overview](#overview)   
- [Usage](#usage)   
  - [The HasSlug Trait](#the-hasslug-trait)   
  - [The SlugOptions Class](#the-slugoptions-class)   
- [How To](#how-to)

# Overview

This feature provides a trait that will generate a unique slug when saving an Eloquent model.   
   
The slug will be automatically saved along with the model using the `creating` and `updating` Eloquent events.

# Usage

### The HasSlug Trait

Your models should use the `App\Traits\HasSlug` trait and implement the `getSlugOptions()` method. 

```php
use App\Traits\HasSlug;
use App\Options\SlugOptions;

class YourModel
{
    use HasSlug;

    /**
     * Get the options for the HasSlug trait.
     *
     * @return SlugOptions
     */
    public static function getSlugOptions()
    {
        return SlugOptions::instance()
            ->generateSlugFrom('name')
            ->saveSlugTo('slug');
    }
}
```

Please note that defining the `getSlugOptions()` method is mandatory.   
If you fail in doing so, the code will throw an error indicating this to you.   
    
The `getSlugOptions()` should always return an `App\Options\SlugOptions` instance.   
You can view all options and their methods of implementation inside the `App/Options/SlugOptions.php` file.
   
However, please remember that you are required to specify the model field from where to generate a slug, as well as the field in which to save the slug.   
   
You can do this using the `generateSlugFrom()` and `saveSlugTo()` methods.   
   
If you fail in implementing these methods, the code will throw an error indicating this to you.   
   
### The SlugOptions Class

> With the help of the `getSlugOptions()` method defined on your given models, you can customize the trait's behavior.   

Use multiple fields as the basis for a slug.   
You can also pass a callable to the `generateSlugsFrom()` method.

```php
public function getSlugOptions()
{
    return SlugOptions::create()
        ->generateSlugsFrom(['first_name', 'last_name'])
        ->saveSlugsTo('slug');
}
```

Allow creation of duplicate slugs.   
By default, the slugs will be unique.

```php
public function getSlugOptions()
{
    return SlugOptions::create()
        ->generateSlugsFrom('name')
        ->saveSlugsTo('slug')
        ->allowDuplicateSlugs();
}
```

Disable the creation of a slug when creating a model.

```php
public function getSlugOptions()
{
    return SlugOptions::create()
        ->generateSlugsFrom('name')
        ->saveSlugsTo('slug')
        ->doNotGenerateSlugOnCreate();
}
```

Disable updating the slug when the model itself is updated.

```php
public function getSlugOptions()
{
    return SlugOptions::create()
        ->generateSlugsFrom('name')
        ->saveSlugsTo('slug')
        ->doNotGenerateSlugOnUpdate();
}
```

# How To

Override the generated slug by setting it to another value than the generated slug.   
   
Don't forget to `save()` the model to persist the change to your database.

```php
$model = Model(['name' => 'My Name']); 
// slug is now "my-name"

$model->slug = 'my-modified-name';
// slug is now "my-modified-name"

$model-save();
// the slug change has now been persisted to the database
```

If you want to explicitly update the slug on the model, you can call the `generateSlug()` method on your model at any time to make the slug according to your other options.   
   
Don't forget to `save()` the model to persist the change to your database.

```php
// load a model instance
$model = YourModel::find($id);

// sync the slug
$model->generateSlug();

// persist the change to the database
$model-save();
```