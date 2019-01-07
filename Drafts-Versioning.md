- [Overview](#overview)   
- [Usage](#usage)   
  - [The Draft Model](#the-draft-model)   
  - [The Migration](#the-migration)   
  - [The HasDrafts Trait](#the-hasdrafts-trait)   
  - [The DraftOptions Class](#the-draftoptions-class)   
  - [The Eloquent Model Events](#the-eloquent-model-events)
  - [The Global Query Scope](#the-global-query-scope)
- [How To](#how-to)   
  - [Basic Implementation](#basic-implementation)   
  - [Admin Implementation](#admin-implementation)

# Overview

This feature provides a trait that allows you to easily create `draft instances` of your Eloquent model records.   
   
Every `draft instance` for an `existing model record` is stored inside the `drafts database table` by default, under the form of a JSON string representing the model's entire structure, including relations.   
   
When creating a `draft instance` of a `non-existing model record`, instead of storing that draft inside the `drafts table`, it simply creates the model record, populating the `drafted_at column` that should be present on the model's table, to the current date.   
   
This type of draft, is called `limbo draft`, because it  exists as an individual entity, and doesn't yet belong to any existing model record.

# Usage

### The Draft Model

Drafts are managed by the `App\Models\Version\Draft` model.   
   
Below you'll find some basic functionalities of the `App\Models\Version\Draft`, but you can always inspect the model's functionalities yourself, or even add some new features to it.   
   
Get the `original model` for a draft.   
   
The `App\Models\Version\Draft` model is `polymorphically related` to the models, using a `belongs to` relation called `draftable`.

```php
$draft->draftable // returns a model instance
```

Get the `user` that created the draft.   
   
The `App\Models\Version\Draft` model defines a `one to many` relation to the `App\Models\Auth\User` model, called `user`.

```php
$draft->user // returns an instance of the "User" model
```

Get the `drafts` for a given model.

```php
Draft::whereDraftable($model->id, get_class($model))->get()
```

### The Migration

For each `model` you want to enable the `draft` functionality, you need to add a `new column` inside it's `database table`.   
   
This will create a column called `drafted_at` of type `timestamp`.   
This column will be used to know if a `model record` is actually a `limbo draft`.

```php
Schema::create('your_model_table', function (Blueprint $table) {
    ...
    Draft::column($table);
    ...
});
```

### The HasDrafts Trait

Your models should use the `App\Traits\HasDrafts` trait and implement the `getDraftOptions()` method.

```php
use App\Traits\HasDrafts;
use App\Options\DraftOptions;

class YourModel
{
    use HasDrafts;

    /**
     * Get the options for the HasDrafts trait.
     *
     * @return DraftOptions
     */
    public static function getDraftOptions()
    {
        return DraftOptions::instance();
    }
}
```

Please note that defining the `getDraftOptions()` method is mandatory.   
If you fail in doing so, the code will throw an error indicating this to you.   
    
The `getDraftOptions()` should always return an `App\Options\DraftOptions` instance.   
   
You can view all options and their methods of implementation inside the `App/Options/DraftOptions.php` file.   

### The DraftOptions Class

> With the help of the `getDraftOptions()` method defined on your given models, you can customize the trait's behavior.   
   
Specify the `model's fields` that can be draftable.   
By default `all fields` are draftable.

```php
/**
 * Get the options for the HasDrafts trait.
 *
 * @return DraftOptions
 */
public static function getDraftOptions()
{
    return DraftOptions::instance()
        ->fieldsToDraft('name', 'content');
}
```

Specify a `model's relations` to be drafted alongside the model itself.   
By default `no relations` are drafted automatically.

```php
/**
 * Get the options for the HasDrafts trait.
 *
 * @return DraftOptions
 */
public static function getDraftOptions()
{
    return DraftOptions::instance()
        ->relationsToDraft('children', 'other-relation');
}
```

Disable `deleting a draft` once it's `published`.   
By default, when publishing a draft, the entry from the `drafts database table` is removed.

```php
/**
 * Get the options for the HasDrafts trait.
 *
 * @return DraftOptions
 */
public static function getDraftOptions()
{
    return DraftOptions::instance()
        ->doNotDeletePublishedDrafts();
}
```

Disable `creating a revision` for the `published draft`.   
By default, when publishing a draft for a model, if that model supports the `revisioning functionality`, a new revision will be created for that model, containing the data that was considered original, before the draft was published.

```php
/**
 * Get the options for the HasDrafts trait.
 *
 * @return DraftOptions
 */
public static function getDraftOptions()
{
    return DraftOptions::instance()
        ->disableRevisioningWhenPublishingADraft();
}
```

Enable `soft relation drafting`.   
By default this is set to false.   
   
The `soft relation drafting` is a concept stating that when `saving a draft with relations`, if for a `draftable relation` no data is provided, the draft will be saved with this relation's original data, instead of saving it empty.

```php
/**
 * Get the options for the HasDrafts trait.
 *
 * @return DraftOptions
 */
public static function getDraftOptions()
{
    return DraftOptions::instance()
        ->softRelationDrafting();
}
```

### The Eloquent Model Events

In the drafting process of a model, `2 Eloquent events` are fired:
- `drafting` - fired before the drafting process
- `drafted` - fired after the drafting process
   
You can use these 2 events, just like any other Eloquent events (`saving`, `saved`, etc.) if your application logic demands it.

### The Global Query Scope

Once the `App\Traits\HasDrafts` trait has been applied on a `model`, the `App\Scopes\DraftingScope` global query scope has also been applied automatically from the `bootHasDrafts()` method on the trait.   
   
The main thing to note here, is that with the `App\Scopes\DraftingScope` applied, when querying for model records, using the usual Laravel's way, the `limbo drafts` meaning the records that have inside the `drafted_at column` anything other than `NULL` will be ignored by default.   
   
```php
// "limbo drafts" will be ignored from the result no matter what
YourModel::where(...)->get(); 
```

Fetch all model records, `including limbo drafts`.

```php
// apply the "withDrafts" query scope defined in the "App\Scopes\DraftingScope"
YourModel::withDrafts()->where(...)->get(); 
```

Fetch the model records, `excluding limbo drafts` ones.

```php
// apply the "withoutDrafts" query scope defined in the "App\Scopes\DraftingScope"
YourModel::withoutDrafts()->where(...)->get(); 

// or simply query the normal way
YourModel::where(...)->get(); 
```

Fetch `only limbo drafts` model records.

```php
// apply the "onlyDrafts" query scope defined in the "App\Scopes\DraftingScope"
YourModel::onlyDrafts()->where(...)->get(); 
```

# How To

### Basic Implementation

`Save` a draft.

```php
// if the model record doesn't yet exist inside the database
// a "limbo draft" will be created
$model = new YourModel();
$model->saveAsDraft(request()->all());

// if the model record exists inside the database
// a "draft" will be created for that record
$model = YourModel::find($id);
$model->saveAsDraft(request()->all());

// to update an existing draft record
// pass the "Draft" model instance as the second argument
$model = YourModel::find($id);
$draft = Draft::find($id);

$model->saveAsDraft(request()->all(), $draft);
```

`Publish` a draft.

```php
$model = YourModel::find($id);
$draft = Draft::find($id);

$model->publishDraft($draft);

// to publish a "limbo draft"
// don't pass the "Draft" model instance as the first argument
$model = YourModel::find($id);
$model->publishDraft();
```

`Delete` drafts.

```php
// to delete a single draft
// call the "deleteDraft()" method with the "draft id" as its argument
$model = YourModel::find($id);
$model->deleteDraft($draftId);

// to delete all drafts related to a model instance
// call the "deleteAllDrafts()" method
$model = YourModel::find($id);
$model->deleteAllDrafts();
```

Get the drafts belonging to a model.   
   
The `App\Traits\HasDrafts` defines a `polymorphically one to many` relation to the `App\Models\Version\Draft` model, called `drafts`.

```php
$model = YourModel::find($id);
$model->drafts; // returns all drafts for the given model instance
```

### Admin Implementation

After you've setup your models to support the drafting feature, you'll want to enable that on your entities, inside the admin panel.   
   
You can easily accomplish that by following the below steps.  
   
> Please note that the full example below assumes the following:   
> - you're using the `App\Traits\CanCrud` trait on your `admin crud controllers`   
> - you're using the `button()` helper to display `draft buttons`   
> - you're using the same `blade views structure` for `crud` already defined in the other existing entities   
>   
> Although not required, following these 3 suggestions will drastically ease your implementation of the drafting functionality.   
>   
> If you want, you can use your own methods, but be aware that you might have to write additional logic, different from what's described below.
   
In your `routes/web.php` file add the following routes for managing the draft operations.

```php
// handles the drafts listing
Route::get('drafts', ['as' => 'admin.your_entity.drafts', 'uses' => 'YourController@drafts', 'permissions' => 'drafts-list']);

// handles a single draft view / edit
Route::get('draft/{draft}', ['as' => 'admin.your_entity.draft', 'uses' => 'YourController@draft', 'permissions' => 'drafts-publish']);

// handles a single limbo draft view / edit
Route::match(['get', 'put'], 'limbo/{id}', ['as' => 'admin.your_entity.limbo', 'uses' => 'YourController@limbo', 'permissions' => 'drafts-save']);
```

Inside your `controller` file, add the following methods, corresponding to each `route` defined above.

```php
// the "drafts" method
// used for displaying a list of "limbo drafts"

/**
 * @param \Illuminate\Http\Request $request
 * @return \Illuminate\View\View
 */
public function drafts(Request $request)
{
    return $this->_drafts(function () use ($request, $filter, $sort) {
        $this->items = YourModel::onlyDrafts()->paginate(10);
        $this->title = 'Your Drafted Entities Title';
        $this->view = view('admin.your_entity_view_path.drafts');
        $this->vars = [
            // some additional variables that your "drafts" view will use
        ];
    });
}

// the "draft" method
// used for viewing / editing a model instance's "draft"

/**
 * @param \App\Models\Version\Draft $draft
 * @return \Illuminate\View\View
 */
public function draft(Draft $draft)
{
    return $this->_draft(function () use ($draft) {
        $this->item = $draft->draftable;
        $this->item->publishDraft($draft);

        $this->title = 'Your Entity Draft Title';
        $this->view = view('admin.your_entity_view_path.draft');
        $this->vars = [
            // some additional variables that your "draft" view will use
        ];
    }, $draft);
}

// the "limbo" method
// used for viewing / editing a "limbo draft"

/**
 * @param \Illuminate\Http\Request $request
 * @param int $id
 * @return \Illuminate\Http\RedirectResponse
 * @throws Exception
 */
public function limbo(Request $request, $id)
{
    return $this->_limbo(function () {
        $this->title = 'Your Entity Limbo Draft Title';
        $this->view = view('admin.your_entity_view_path.limbo');
        $this->vars = [
            // some additional variables that your "limbo" view will use
        ];
    }, function () use ($request) {
        $this->item->saveAsDraft($request->all());
        $this->redirect = redirect()->route('admin.your_entity_view_path.drafts');
    }, $id, $request, new YourEntityRequest());
}
```

In your entity's `index.blade.php` file, inside the `footer section` add the `button` for directing the admin to the `list of limbo drafted records`.

```html
@section('footer')
    <section class="actions">
        <!-- pass the route to your controller's "drafts" method as the first parameter -->
        {!! button()->draftedRecords(route('admin.your_entity.drafts')) !!}
        ...
    </section>
@endsection
```

In your entity's `_buttons.blade.php` file add the `button` for `saving a model instance` as a `draft` or `limbo draft`. 

```html
<section class="actions">
    <!-- leave the parameter exactly as is -->
    <!-- the button will point to an already defined route -->
    {!! button()->saveAsDraft(route('admin.drafts.save')) !!}
    ...
</section>
```

In your entity's `_tabs.blade.php` file, add the `drafts tab` to be able to navigate to the `drafts list` when `editing` e model record.

```php
@if($item->exists && !isset($on_draft) && !isset($on_limbo_draft) && !isset($on_revision))
    {!! draft()->tab($item, 'admin.your_entity.draft') !!}
@endif
```

> Please note that the `drafts tab` should be shown only when:   
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

In your entity's `_form.blade.php` file, right after opening the form, add the following `hidden inputs` for the drafting functionality to know how to parse it's `save requests`.

```html
<!-- the "_class" field containing your model's class as string -->
{!! form()->hidden('_class', \App\Models\YourModel::class) !!}

<!-- the "_request" field, containing your entity's form request class as string -->
{!! form()->hidden('_request', \App\Http\Requests\YourRequest::class) !!}

<!-- the "_id" field containing your existing model's id, or null -->
{!! form()->hidden('_id', $item->exists ? $item->id : null) !!}

<!-- the "_back" field containing your entity's route for "limbo drafts" -->
<!-- this will be used to know where to redirect after saving an existing "limbo draft" -->
{!! form()->hidden('_back', route('admin.your_entity.drafts')) !!}

...
```

In your entity's `_form.blade.php` file, right before closing the form, add the `drafts container`, responsible for displaying the `drafts list` for a model instance.

```php
@if($item->exists && !isset($on_draft) && !isset($on_limbo_draft) && !isset($on_revision))
    {!! draft()->container($item) !!}
@endif
```

> Please note that the `drafts container` should be shown only when:   
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

Inside your entity's `views directory`, create a new file called `drafts.blade.php`, responsible for listing the `limbo drafts` for the given model.
   
```html
@extends('layouts::admin.default')

@section('content')
    ...
    <section class="list">
        <table cellspacing="0" cellpadding="0" border="0">
            <thead>
                <tr>
                    ...
                    <td class="actions-drafted">Actions</td>
                </tr>
            </thead>
            <tbody>
            @if($items->count() > 0)
                @foreach($items as $index => $item)
                    <tr class="{!! $index % 2 == 0 ? 'even' : 'odd' !!}">
                        ...
                        <td>
                            {!! button()->publishLimboDraft(route('admin.drafts.publish_limbo'), $item) !!}
                            {!! button()->editRecord(route('admin.your_entity.limbo', $item->id)) !!}
                            {!! button()->deleteLimboDraft(route('admin.drafts.delete_limbo'), $item) !!}
                        </td>
                    </tr>
                @endforeach
            @else
                ...
            @endif
            </tbody>
        </table>
    </section>
@endsection

@section('footer')
    ...
    <section class="actions">
        {!! button()->goBack(route('admin.your_entity.index')) !!}
    </section>
@endsection
```

> This above view should be somewhat similar to your `index.blade.php`.     
>    
> The only `required differences` are:
> - for your `actions <td>` use the `actions-drafted` class   
> - use the following buttons: `publishLimboDraft`, `editRecord`, `deletLimboDraft`   
> - inside the `footer section` specify the `goBack` button

Inside your entity's `views directory`, create a new file called `draft.blade.php`.

```html
@extends('layouts::admin.default')

@section('content')
    <section class="tabs">
        <!-- specify "on_draft = true" -->
        <!-- for the "_tabs" logic to take it into consideration -->
        @include('admin.your_entity_view_path._tabs', ['on_draft' => true])
    </section>

    <section class="view">
        <!-- specify "on_draft = true" -->
        <!-- for the "_form" logic to take it into consideration -->
        @include('admin.your_entity_view_path._form', ['on_draft' => true])
    </section>
@endsection

<!-- pass the loaded draft model as the first parameter -->
<!-- pass your loaded model as the second parameter -->
{!! draft()->view($draft, $item) !!}
```

Inside your entity's `views directory`, create a new file called `limbo.blade.php`.

```html
@extends('layouts::admin.default')

@section('content')
    <section class="tabs">
        <!-- specify "on_limbo_draft = true" -->
        <!-- for the "_tabs" logic to take it into consideration -->
        @include('admin.your_entity_view_path._tabs', ['on_limbo_draft' => true])
    </section>

    <section class="view">
        <!-- specify "on_limbo_draft = true" -->
        <!-- for the "_tabs" logic to take it into consideration -->
        @include('admin.your_entity_view_path._form', ['on_limbo_draft' => true])
    </section>
@endsection

@section('footer')
    <section class="actions left">
        <!-- Specify your entity's "limbo draft list route" as the first parameter -->
        {!! button()->cancelAction(route('admin.your_entity.drafts')) !!}
    </section>
    <section class="actions">
        {!! button()->saveRecord(['style' => 'margin-right: 5px;']) !!}
        {!! button()->publishDraft(route('admin.drafts.publish_limbo')) !!}
    </section>
@endsection
```