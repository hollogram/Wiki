- [Overview](#overview)   
- [Usage](#usage)   
  - [The HasUrl Trait](#the-hasurl-trait)   
  - [The UrlOptions Class](#the-urloptions-class)    
- [How To](#how-to)


# Overview

This feature provides a trait that will generate a unique url when saving an Eloquent model.   
   
The url will be automatically saved inside the `urls` table, along with the model using the `created` and `updated` Eloquent events.   
   
When the model is deleted, the referencing url will also be deleted.

# Usage

### The HasUrl Trait

Your models should use the `App\Traits\HasUrl` trait and implement the `getUrlOptions()` method.   
> Please note that the `HasUrl` trait is using at its turn, the `HasSlug` trait.

```php
use App\Traits\HasUrl;
use App\Options\UrlOptions;

class YourModel
{
    use HasUrl;

    /**
     * Get the options for the HasUrl trait.
     *
     * @return UrlOptions
     */
    public static function getUrlOptions()
    {
        return UrlOptions::instance()
            ->routeUrlTo('Your\Full\Namespaced\Controller', 'yourControllerMethod')
            ->generateUrlSlugFrom('name')
            ->saveUrlSlugTo('slug');
    }
}
```

Please note that defining the `getUrlOptions()` method is mandatory.   
If you fail in doing so, the code will throw an error indicating this to you.   
    
The `getUrlOptions()` should always return an `App\Options\UrlOptions` instance.   
You can view all options and their methods of implementation inside the `App/Options/UrlOptions.php` file.
   
However, please remember that you are required to specify the routing from where Laravel will dispatch it's route requests, as well as the model field from where to generate a slug and the field in which to save the slug.   
   
You can do this using the `routeUrlTo()`, `generateUrlSlugFrom()` and `saveUrlSlugTo()` methods.   
   
If you fail in implementing this methods, the code will throw an error indicating this to you.   
   
### The UrlOptions Class

> With the help of the `getUrlOptions()` method defined on your given models, you can customize the trait's behavior.   

Set a prefix value for the url.   
The `prefixUrlWith()` method accepts a string, array or callable as its parameter.

```php
public function getUrlOptions()
{
    return UrlOptions::create()
            ->routeUrlTo('Your\Full\Namespaced\Controller', 'yourControllerMethod')
            ->generateUrlSlugFrom('name')
            ->saveUrlSlugTo('slug')
            ->prefixUrlWith(function ($prefix, $model) {
                foreach ($model->ancestors()->get() as $parent) {
                    $prefix[] = $parent->slug;
                }

                return implode('/' , (array)$prefix);
            });
}
```

Set a suffix value for the url.   
The `suffixUrlWith()` method accepts a string, array or callable as its parameter.

```php
public function getUrlOptions()
{
    return UrlOptions::create()
            ->routeUrlTo('Your\Full\Namespaced\Controller', 'yourControllerMethod')
            ->generateUrlSlugFrom('name')
            ->saveUrlSlugTo('slug')
            ->suffixUrlWith(['my', 'suffix']);
}
```

Disable updating urls in cascade.   
By default, when updating a url, all of its children urls identified based on the old format, will be updated to the new format.

```php
public function getUrlOptions()
{
    return UrlOptions::create()
            ->routeUrlTo('Your\Full\Namespaced\Controller', 'yourControllerMethod')
            ->generateUrlSlugFrom('name')
            ->saveUrlSlugTo('slug')
            ->doNotUpdateCascading();
}
```

# How To

Get the url instance for a model.   
The url is polymorphically related to the model using a one-to-one relation called `url()`.   
Also, this relation is eager-loaded when fetching a model.

```php
$model = Model::find(id);
$url = $model->url;
```

Get the model instance for a url.   
The model is polymorphically related to the url using the inverse one-to-one relation called `urlable()`.

```php
$url = Url::where('url', 'my-url')->first();
$model = $url->urlable;
```

Manually disable url generation for a save event.

```php
app(Model::class)->doNotGenerateUrl()->create(['name' => 'My Name']);

// or

$model = Model::find(id);
$model->doNotGenerateUrl()->update(['name' => 'My Name']);
```