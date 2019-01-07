- [Overview](#overview)   
- [Usage](#usage)   
  - [The Revision Model](#the-revision-model)   
  - [The HasRevisions Trait](#the-hasrevisions-trait)   
  - [The RevisionOptions Class](#the-revisionoptions-class)   
  - [The Eloquent Model Events](#the-eloquent-model-events)
- [How To](#how-to)   
  - [Basic Implementation](#basic-implementation)   
  - [Admin Implementation](#admin-implementation)   

# Overview

This feature provides a trait that allows you to easily create `revision instances` of your Eloquent model records.   
   
Every `revision instance` is stored inside the `revisions database table` by default, under the form of a JSON string representing the model's entire structure, including relations.

# Usage

### The Revision Model

Revisions are managed by the `App\Models\Version\Revision` model.   
   
Below you'll find some basic functionalities of the `App\Models\Version\Revision`, but you can always inspect the model's functionalities yourself, or even add some new features to it.   
   
Get the `original model` for a revision.   
   
The `App\Models\Version\Revision` model is `polymorphically related` to the models, using a `belongs to` relation called `revisionable`.

```php
$revision->revisionable// returns a model instance
```

Get the `user` that created the revision.   
   
The `App\Models\Version\Revision` model defines a `one to many` relation to the `App\Models\Auth\User` model, called `user`.

```php
$revision->user // returns an instance of the "User" model
```

Get the `revisions` for a given model.

```php
Revision::whereRevisionable($model->id, get_class($model))->get()
```

### The HasRevisions Trait

Your models should use the `App\Traits\HasRevisions ` trait and implement the `getRevisionOptions()` method.

```php
use App\Traits\HasRevisions ;
use App\Options\RevisionOptions;

class YourModel
{
    use HasRevisions ;

    /**
     * Get the options for the HasRevisions trait.
     *
     * @return RevisionOptions
     */
    public static function getRevisionOptions()
    {
        return RevisionOptions::instance();
    }
}
```

Please note that defining the `getRevisionOptions()` method is mandatory.   
If you fail in doing so, the code will throw an error indicating this to you.   
    
The `getRevisionOptions()` should always return an `App\Options\RevisionOptions` instance.   
   
You can view all options and their methods of implementation inside the `App/Options/RevisionOptions.php` file.   

### The RevisionOptions Class

> With the help of the `getRevisionOptions()` method defined on your given models, you can customize the trait's behavior.   
   
Enable `creating a revision` when the model is `created`.   
By default, revisions are created only when the model is `updated`.

```php
/**
 * Get the options for the HasRevisions trait.
 *
 * @return RevisionOptions
 */
public static function getRevisionOptions()
{
    return RevisionOptions::instance()
        ->enableRevisionOnCreate();
}
```

Set a `limit` for the number of revisions that are created.   
By default, no limit is specified, allowing for infinite revisions to be created.   

However, it's recommended to limit created revisions, because very quickly, you can end up with a very large number of revisions.   
   
By limiting the revisions, when creating a new revision, and the limit is reached, the oldest revision will be deleted, making room for the new revision.

```php
/**
 * Get the options for the HasRevisions trait.
 *
 * @return RevisionOptions
 */
public static function getRevisionOptions()
{
    return RevisionOptions::instance()
        ->limitRevisionsTo(100);
}
```

Specify the `model's fields` that can be revisionable.   
By default `all fields` are revisionable.

```php
/**
 * Get the options for the HasRevisions trait.
 *
 * @return RevisionOptions
 */
public static function getRevisionOptions()
{
    return RevisionOptions::instance()
        ->fieldsToRevision('name', 'content');
}
```

Specify a `model's relations` to be revisioned alongside the model itself.   
By default `no relations` are revisioned automatically.

```php
/**
 * Get the options for the HasRevisions trait.
 *
 * @return RevisionOptions
 */
public static function getRevisionOptions()
{
    return RevisionOptions::instance()
        ->relationsToRevision('children', 'other-relation');
}
```

Disable `creating revisions` when `rolling back` an old revision.   
By default this is enabled, meaning that when you roll back an existing revision, a new revision containing the model record's original data will be created.

```php
/**
 * Get the options for the HasRevisions trait.
 *
 * @return RevisionOptions
 */
public static function getRevisionOptions()
{
    return RevisionOptions::instance()
        ->disableRevisioningWhenRollingBack();
}
```

### The Eloquent Model Events

In the revisioning process of a model, `2 Eloquent events` are fired:
- `revisioning` - fired before the revisioning process
- `revisioned` - fired after the revisioning process
   
You can use these 2 events, just like any other Eloquent events (`saving`, `saved`, etc.) if your application logic demands it.

# How To

### Basic Implementation

`Create` a new revision for a model instance.

```php
// the general way of creating a revision
$model = YourModel::find($id);
$model->createNewRevision();

// but you can also create a revision
// overruling the Eloquent model events
$model = YourModel::find($id);->saveAsRevision();
```

`Rollback` a model instance to an older revision.

```php
$model = YourModel::find($id);
$revision = Revision::find($id);

$model->rollbackToRevision($revision);
```

`Delete` all revisions for a model instance.

```php
$model = YourModel::find($id);
$model->deleteAllRevisions();
```

`Delete` only the oldest revision that exceed the limit.

```php
$model = YourModel::find($id);
$model->clearOldRevisions();
```

### Admin Implementation

After you've setup your models to support the revisioning feature, you'll want to enable that on your entities, inside the admin panel.   
   
You can easily accomplish that by following the below steps.  
   
> Please note that the full example below assumes the following:   
> - you're using the `App\Traits\CanCrud` trait on your `admin crud controllers`   
> - you're using the `button()` helper to display `revision buttons`   
> - you're using the same `blade views structure` for `crud` already defined in the other existing entities   
>   
> Although not required, following these 3 suggestions will drastically ease your implementation of the revisioning functionality.   
>   
> If you want, you can use your own methods, but be aware that you might have to write additional logic, different from what's described below.

In your `routes/web.php` file add the following route for managing the revision operations.

```php
// handles a single revision view / edit
Route::get('revision/{revision}', ['as' => 'admin.your_entity.revision', 'uses' => 'YourController@revision', 'permissions' => 'revisions-rollback']);
```

Inside your `controller` file, add the following method, corresponding to the `route` defined above.

```php
/**
 * @param \App\Models\Version\Revision $revision
 * @return \Illuminate\View\View
 */
public function revision(Revision $revision)
{
    return $this->_revision(function () use ($revision) {
        $this->item = $revision->revisionable;
        $this->item->rollbackToRevision($revision);

        $this->title = 'Your Entity Revision Title';
        $this->view = view('admin.your_entity_view_path.revision');
        $this->vars = [
            // some additional variables that your "revision" view will use
        ];
    }, $revision);
}
```

In your entity's `_tabs.blade.php` file, add the `revisions tab` to be able to navigate to the `revisions list` when `editing` e model record.

```php
@if($item->exists && !isset($on_draft) && !isset($on_limbo_draft) && !isset($on_revision))
    {!! revision()->tab($item, 'admin.your_entity.revision') !!}
@endif
```

> Please note that the `revisions tab` should be shown only when:   
> - the model exists   
> - we're not actually viewing / editing a draft   
> - we're not actually viewing / editing a limbo draft   
> - we're not actually viewing / editing a revision 

In your entity's `_form.blade.php` file, at the very beginning, replace the `form opening` logic with.

```php
@if($item->exists)
    @if(isset($on_draft) || isset($on_limbo_draft) || isset($on_revision))
        {!! form_admin()->model($item, ['method' => isset($on_draft) || isset($on_revision) ? 'POST' : 'PUT','class' => 'form', 'files' => true]) !!}
    @else
        {!! form_admin()->model($item, ['url' => $url, 'method' => 'PUT', 'class' => 'form', 'files' => true]) !!}
    @endif
@else
    {!! form_admin()->open(['url' => $url, 'method' => 'POST', 'class' => 'form', 'files' => true]) !!}
@endif
```

In your entity's `_form.blade.php` file, right before closing the form, add the `revisions container`, responsible for displaying the `revisions list` for a model instance.

```php
@if($item->exists && !isset($on_draft) && !isset($on_limbo_draft) && !isset($on_revision))
    {!! revision()->container($item) !!}
@endif
```

> Please note that the `revisions container` should be shown only when:   
> - the model exists   
> - we're not actually viewing / editing a draft   
> - we're not actually viewing / editing a limbo draft   
> - we're not actually viewing / editing a revision   
   
In your entity's `_form.blade.php` file, usually at the very end, condition your `form request` to fire only for the original entity and not when `on draft` or `on revision`. Otherwise you'll get errors.

```php
@if(!isset($on_draft) && !isset($on_limbo_draft) && !isset($on_revision))
    @section('bottom_scripts')
        {!! JsValidator::formRequest(App\Http\Requests\YourRequest::class, '.form') !!}
    @append
@endif
```

Inside your entity's `views directory`, create a new file called `revision.blade.php`.

```html
@extends('layouts::admin.default')

@section('content')
    <section class="tabs">
        <!-- specify "on_revision = true" -->
        <!-- for the "_tabs" logic to take it into consideration -->
        @include('admin.your_entity_view_path._tabs', ['on_revision' => true])
    </section>

    <section class="view">
        <!-- specify "on_revision = true" -->
        <!-- for the "_form" logic to take it into consideration -->
        @include('admin.your_entity_view_path._form', ['on_revision' => true])
    </section>
@endsection

<!-- pass the loaded revision model as the first parameter -->
<!-- pass your loaded model as the second parameter -->
{!! revision()->view($revision, $item) !!}
```