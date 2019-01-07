- [Overview](#overview)   
- [Usage](#usage)   

# Overview

This is an extended feature of the [[Crud]] functionality.   
It provides an easy way to implement a `preview function` for an entity, using the `App\Traits\CanCrud` trait.   
   
> Please note that you can also implement this feature without the `App\Traits\CanCrud` trait, but you will have to write additional code than provided above.

# Usage

You can easily enable the `preview` functionality on your entities, inside the `admin` in 3 easy steps.   
   
> Please note that the full example below assumes the following:   
> - you're using the `App\Traits\CanCrud` trait on your `admin crud controllers`   
> - you're using the `button()` helper to display `draft buttons`   
> - you're using the same `blade views structure` for `crud` already defined in the other existing entities   
>   
> Although not required, following these 3 suggestions will drastically ease your implementation of the drafting functionality.   
>   
> If you want, you can use your own methods, but be aware that you might have to write additional logic, different from what's described below.   
   
In `routes/web.php` add the route responsible for managing your entity's preview logic.

```php
Route::match(['post', 'put'], 'preview/{your_model?}', ['as' => 'admin.your_entity.preview', 'uses' => 'YourController@preview', 'permissions' => 'your-entity-preview']);
```

In your `controller file`, add the `preview()` method, corresponding to the route defined above.   
You can read more about the `_preview()` method on the [[CRUD]] documentation page.

```php
/**
 * @param YourRequest $request
 * @param YourModel|null $page
 * @return \Illuminate\Http\RedirectResponse
 * @throws Exception
 */
public function preview(YourRequest $request, YourModel $model = null)
{
    return $this->_preview(function () use ($request, $model) {
        if ($model&& $model->exists) {
            $this->item = $model;
            $this->item->update($request->all());
        } else {
            $this->item = YourModel::create($request->all());
        }
    });
}
```

Finally, in your entity's `_buttons.blade.php` file, add the `preview button` to enable the admin to access that functionality.

```html
<section class="actions">
    {!! button()->previewRecord(route('admin.your_entity.preview', $item->id)) !!}
</section>
```

As an extra step, if it's not done by default.   
You every `front layout` should contain the following at the bottom.

```php
@php preview()->handle(); @endphp
```
> This is the `preview handler` that checks if the page is displayed for preview purposes or not. If it is, then it means there's an `opened database transaction` from when saving the model record for preview. The `preview handler` does nothing but `roll back` that transaction, so your model record is left intact.