- [Overview](#overview)   
- [Usage](#usage)   
  - [The CanCrud Trait](#the-cancrud-trait)   
  - [The Basic Crud Methods](#the-basic-crud-methods)   
  - [The Advanced Crud Methods](#the-advanced-crud-methods)   
- [Observations](#observations)

# Overview

This feature provides a trait that allows you to easily setup a `CRUD` (create-read-update-delete) system for your entities.   
   
The `App\Traits\CanCrud` trait takes care of all the boiler-plate a crud functionality may present.

# Usage

### The CanCrud Trait

Your controllers should use the `App\Traits\CanCrud` trait and define a protected property called `$model`.

```php
use App\Http\Controllers\Controller;
use App\Traits\CanCrud;

class YourController extends Controller
{
    use CanCrud;

    /**
     * @var string
     */
    protected $model = YourModel::class;

    ...
}
```

### The Basic Crud Methods

The `App\Traits\CanCrud` covers the `crud methods` that your `resource controller` might use.

**The `_index()` method**

This method is used for showing the `list view`.   
This method accepts only `GET` requests, so your route should be defined accordingly.

```php
// in your entity crud controller

/**
 * @param Request $request
 * @return \Illuminate\View\View
 */
public function yourListMethod(Request $request)
{
    return $this->_index(function () use ($request) {
        $this->items = YourModel::all();
        $this->title = 'Your Title';
        $this->view = view('your_list_view');
        $this->vars = [
            'types' => 'My Types',
            'options' => 'My Options',
        ];
    });
}
```
> It's mandatory for `your list method` to only return an instance of the `_index()` method.   
> The entire logic should happen inside the `callback` of the `_index()` method.   
>    
> **Required actions:**   
> - Set the `items` property.   
>   - Once you've done that, you can access your `items` from your `view file`, by accessing the `$items` variable.   
> - Set the `view` property.   
>   - This way, the controller will know which `view file` to return for your `list` operation.   
>    
> **Optional actions**   
> - Set the `title` property.   
>   - The value of the title will be appended to your page's `meta title` and it will also be available to you inside your `view file`, by accessing the `$title` variable.   
> - Append to the `vars` property.   
>   - By default, the `_index()` will automatically assign the `items` and `title` to the view, but you can also specify other `variables` to use inside your `view file`.   
> - Any other `custom logic` that your application might require.

**The `_create()` method**

This method is used for showing the `add view`.   
This method accepts only `GET` requests, so your route should be defined accordingly.

```php
// in your entity crud controller

/**
 * @return \Illuminate\View\View
 */
public function yourCreateMethod()
{
    return $this->_create(function () {
        $this->title = 'Add Entity';
        $this->view = view('your_add_view');
        $this->vars = [
            'types' => 'My Types',
            'options' => 'My Options',
        ];
    });
}
```
> It's mandatory for `your create method` to only return an instance of the `_create()` method.   
> The entire logic should happen inside the `callback` of the `_create()` method.   
>    
> **Required actions:**   
> - Set the `view` property.   
>   - This way, the controller will know which `view file` to return for your `create` operation.   
>    
> **Optional actions**   
> - Set the `title` property.   
>   - The value of the title will be appended to your page's `meta title` and it will also be available to you inside your `view file`, by accessing the `$title` variable.   
> - Append to the `vars` property.   
>   - By default, the `_create()` will automatically assign the `item` (new model instance) and `title` to the view, but you can also specify other `variables` to use inside your `view file`.   
> - Any other `custom logic` that your application might require.

**The `_store()` method**

This method is used for handling your `create operation`.   
This method accepts only `POST` requests, so your route should be defined accordingly.

```php
// in your entity crud controller

/**
 * @param YourRequest $request
 * @return \Illuminate\View\View
 */
public function yourStoreMethod(YourRequest $request)
{
    return $this->_store(function () use ($request) {
        YourModel::create($request->all());

        $this->redirect = redirect()->route('your_list_route');
    });
}
```
> It's mandatory for `your store method` to only return an instance of the `_store()` method.   
> The entire logic should happen inside the `callback` of the `_store()` method.   
>    
> **Required actions:**   
> - Handle your `create logic` for your `model`.
> - Set the `redirect` property.   
>   - This way, the controller will know to which `route` to redirect after the model was successfully `created`.   
>    
> **Optional actions**   
> - Any other `custom logic` that your application might require.

**The `_edit()` method**

This method is used for showing the `edit view`.   
This method accepts only `GET` requests, so your route should be defined accordingly.

```php
// in your entity crud controller

/**
 * @param YourModel $model
 * @return \Illuminate\View\View
 */
public function yourEditMethod(YourModel $model)
{
    return $this->_edit(function () use ($model) {
        $this->item = $model;
        $this->title = 'Edit Entity';
        $this->view = view('your_edit_view');
        $this->vars = [
            'types' => 'My Types',
            'options' => 'My Options',
        ];
    });
}
```
> It's mandatory for `your edit method` to only return an instance of the `_edit()` method.   
> The entire logic should happen inside the `callback` of the `_edit()` method.   
>    
> **Required actions:**   
> - Set the `item` property.   
>   - Once you've done that, you can access your `model` inside your `view file`, by accessing the `$item` variable.   
>    
> - Set the `view` property.   
>   - This way, the controller will know which `view file` to return for your `edit` operation.   
>    
> **Optional actions**   
> - Set the `title` property.   
>   - The value of the title will be appended to your page's `meta title` and it will also be available to you inside your `view file` by accessing the `$title` variable.   
> - Append to the `vars` property.   
>   - By default, the `_edit()` method will automatically assign the `item` and `title` to your view, but you can also specify other `variables` to use inside your `view file`.   
> - Any other `custom logic` that your application might require.

**The `_update()` method**

This method is used for handling `your update operation`.   
This method accepts only `PUT` requests, so your route should be defined accordingly.

```php
// in your entity crud controller

/**
 * @param YourRequest $request
 * @param YourModel $model
 * @return \Illuminate\View\View
 */
public function yourUpdateMethod(YourRequest $request, YourModel $model)
{
    return $this->_update(function () use ($request, $model) {
        $model->update($request->all());

        $this->redirect = redirect()->route('your_edit_route');
    }, $request);
}
```
> It's mandatory for `your update method` to only return an instance of the `_update()` method.   
> The entire logic should happen inside the `callback` of the `_update()` method.   
>    
> **Required actions:**   
> - Handle your `update logic` for your `model`.
> - Set the `redirect` property.   
>   - This way, the controller will know to which `route` to redirect after the model was successfully `updated`.   
>    
> **Optional actions**   
> - Any other `custom logic` that your application might require.

**The `_destroy()` method**

This method is used for handling your `delete operation`.   
This method accepts only `DELETE` requests, so your route should be defined accordingly.

```php
// in your entity crud controller

/**
 * @param YourModel $model
 * @return \Illuminate\View\View
 */
public function yourDestroyMethod(YourModel $model)
{
    return $this->_destroy(function () use ($model) {
        $model->delete();

        $this->redirect = redirect()->route('your_list_route');
    });
}
```
> Is mandatory for `your destroy method` to only return an instance of the `_destroy()` method.   
> The entire logic should happen inside the `callback` of the `_destroy()` method.   
>    
> **Required actions:**   
> - Handle your `delete logic` for your `model`.
> - Set the `redirect` property.   
>   - This way, the controller will know to which `route` to redirect after the model was successfully `deleted`.   
>    
> **Optional actions**   
> - Any other `custom logic` that your application might require.

### The Advanced Crud Methods

**The `_deleted()` method**

This method is used for showing the `soft-deleted items list view`.   
This method accepts only `GET` requests, so your route should be defined accordingly.

```php
// in your entity crud controller

/**
 * @param Request $request
 * @return \Illuminate\View\View
 */
public function yourDeletedMethod(Request $request)
{
    return $this->_deleted(function () use ($request) {
        $this->items = YourModel::onlyTrashed()->get();
        $this->title = 'Your Title';
        $this->view = view('your_deleted_list_view');
        $this->vars = [
            'types' => 'My Types',
            'options' => 'My Options',
        ];
    });
}
```
> It's mandatory for `your deleted method` to only return an instance of the `_deleted()` method.   
> The entire logic should happen inside the `callback` of the `_deleted()` method.   
>    
> **Required actions:**   
> - Set the `items` property.   
>   - Once you've done that, you can access your `items` from your `view file`, by accessing the `$items` variable.   
> - Set the `view` property.   
>   - This way, the controller will know which `view file` to return for your `soft-deleted list` operation.   
>    
> **Optional actions**   
> - Set the `title` property.   
>   - The value of the title will be appended to your page's `meta title` and it will also be available to you inside your `view file`, by accessing the `$title` variable.   
> - Append to the `vars` property.   
>   - By default, the `_deleted()` will automatically assign the `items` and `title` to the view, but you can also specify other `variables` to use inside your `view file`.   
> - Any other `custom logic` that your application might require.

**The `_restore()` method**

This method is used for handling your `restoring operation`.   
This method accepts only `PUT` requests, so your route should be defined accordingly.

```php
// in your entity crud controller

/**
 * @param int $id
 * @return \Illuminate\Http\RedirectResponse
 */
public function yourRestoreMethod($id)
{
    return $this->_restore(function () use ($id) {
        $this->redirect = redirect()->route('your_deleted_list_route');
        
        $item = YourModel::onlyTrashed()->findOrFail($id);
        $item->restore();
    });
}
```
> I'ts mandatory for `your restore method` to only return an instance of the `_restore()` method.   
> The entire logic should happen inside the `callback` of the `_restore()` method.   
>    
> **Required actions:**   
> - Handle your `restore logic` for your `model`.
> - Set the `redirect` property.   
>   - This way, the controller will know to which `route` to redirect after the model was successfully `restored`.   
>    
> **Optional actions**   
> - Any other `custom logic` that your application might require.

**The `_delete()` method**

This method is used for handling your `force delete` operations.   
This method accepts only `DELETE` requests, so your route should be defined accordingly.

```php
// in your entity crud controller

/**
 * @param int $id
 * @return \Illuminate\Http\RedirectResponse
 */
public function yourDeleteMethod($id)
{
    return $this->_delete(function () use ($id) {
        $this->redirect = redirect()->route('your_deleted_list_route');

        $item = YourModel::onlyTrashed()->findOrFail($id);
        $item->forceDelete();
    });
}
```
> I'ts mandatory for `your delete method` to only return an instance of the `_delete()` method.   
> The entire logic should happen inside the `callback` of the `_delete()` method.   
>    
> **Required actions:**   
> - Handle your `force delete logic` for your `model`.
> - Set the `redirect` property.   
>   - This way, the controller will know to which `route` to redirect after the model was successfully `force deleted`.   
>    
> **Optional actions**   
> - Any other `custom logic` that your application might require.

**The `_duplicate()` method**

This method is used for handling your `duplicate` operations.   
This method accepts only `POST` requests, so your route should be defined accordingly.

```php
// in your entity crud controller

/**
 * @param YourModel $model
 * @return \Illuminate\Http\RedirectResponse
 */
public function yourDuplicateMethod(YourModel $model)
{
    return $this->_duplicate(function () use ($model) {
        $duplicatedModel = $model->saveAsDuplicate();
        
        $this->redirect = redirect()->route('your_edit_route', $duplicatedModel->id);
    });
}
```
> I'ts mandatory for `your duplicate method` to only return an instance of the `_duplicate()` method.   
> The entire logic should happen inside the `callback` of the `_duplicate()` method.   
>    
> **Required actions:**   
> - Handle your `duplicate logic` for your `model`.
> - Set the `redirect` property.   
>   - This way, the controller will know to which `route` to redirect after the model was successfully `duplicated`.   
>    
> **Optional actions**   
> - Any other `custom logic` that your application might require.

**The `_preview()` method**

This method is used for handling your `preview` operations.   
This method accepts both `POST` and `PUT` requests, so your route should be defined accordingly.

```php
// in your entity crud controller

/**
 * @param YourRequest $request
 * @param YourModel|null $model
 * @return \Illuminate\Http\RedirectResponse
 */
public function yourPreviewMethod(YourRequest $request, YourModel $model = null)
{
    return $this->_preview(function () use ($request, $model) {
        if ($model && $model->exists) {
            $this->item = $model;
            $this->item->update($request->all());
        } else {
            $this->item = YourModel::create($request->all());
        }
    });
}
```
> I'ts mandatory for `your preview method` to only return an instance of the `_preview()` method.   
> The entire logic should happen inside the `callback` of the `_preview()` method.   
>    
> **Required actions:**   
> - Set the `item` property while `creating` / `updating` the model.   
>   - This is used by the `controller dispatcher` to pass the `model` to the `front-end controller`.   
>    
> **Optional actions**   
> - Any other `custom logic` that your application might require.

**The `_drafts()` method**

This method is used for showing the `drafts list view`.   
This method accepts only `GET` requests, so your route should be defined accordingly.

```php
// in your entity crud controller

/**
 * @param Request $request
 * @return \Illuminate\View\View
 */
public function yourDraftsMethod(Request $request)
{
    return $this->_drafts(function () use ($request) {
        $this->items = YourModel::onlyDrafts->get();
        $this->title = 'Your Title';
        $this->view = view('your_draft_list_view');
        $this->vars = [
            'types' => 'My Types',
            'options' => 'My Options',
        ];
    });
}
```
> It's mandatory for `your draft list method` to only return an instance of the `_drafts()` method.   
> The entire logic should happen inside the `callback` of the `_drafts()` method.   
>    
> **Required actions:**   
> - Set the `items` property.   
>   - Once you've done that, you can access your `items` from your `view file`, by accessing the `$items` variable.   
> - Set the `view` property.   
>   - This way, the controller will know which `view file` to return for your `list` operation.   
>    
> **Optional actions**   
> - Set the `title` property.   
>   - The value of the title will be appended to your page's `meta title` and it will also be available to you inside your `view file`, by accessing the `$title` variable.   
> - Append to the `vars` property.   
>   - By default, the `_index()` will automatically assign the `items` and `title` to the view, but you can also specify other `variables` to use inside your `view file`.   
> - Any other `custom logic` that your application might require.

**The `_draft()` method**

This method is used for showing the `draft edit view`.   
This method accepts only `GET` requests, so your route should be defined accordingly.

```php
// in your entity crud controller

/**
 * @param Draft $draft
 * @return \Illuminate\View\View
 */
public function yourDraftMethod(Draft $draft)
{
    return $this->_draft(function () use ($draft) {
        $this->item = $draft->draftable;
        $this->item->publishDraft($draft);

        $this->title = 'Edit Draft Entity';
        $this->view = view('your_draft_view');
        $this->vars = [
            'types' => 'My Types',
            'options' => 'My Options',
        ];
    }, $draft);
}
```
> It's mandatory for `your draft edit method` to only return an instance of the `_draft()` method.   
> The entire logic should happen inside the `callback` of the `_draft()` method.   
>    
> **Required actions:**   
> - Set the `item` property to the `original model`.   
>   - Once you've done that, you can access your `model` inside your `view file`, by accessing the `$item` variable.   
>    
> - `Publish` the original model's `draft`.   
>   - This is done inside a `database transaction` that's later `rolled back`, but the `publish` needs to happen in order for you to see the `draft model data` inside your `form`.
>    
> - Set the `view` property.   
>   - This way, the controller will know which `view file` to return for your `draft edit` operation.   
>    
> **Optional actions**   
> - Set the `title` property.   
>   - The value of the title will be appended to your page's `meta title` and it will also be available to you inside your `view file` by accessing the `$title` variable.   
> - Append to the `vars` property.   
>   - By default, the `_draft()` method will automatically assign the `item` and `title` to your view, but you can also specify other `variables` to use inside your `view file`.   
> - Any other `custom logic` that your application might require.

**The `_limbo()` method**

This method is used for showing and saving a `limbo draft`.   
This method accepts both `GET` and `PUT` requests, so your route should be defined accordingly.

```php
// in your entity crud controller

/**
 * @param \Illuminate\Http\Request $request
 * @param int $id
 * @return \Illuminate\Http\RedirectResponse
 */
public function yourLimboMethod(Request $request, $id)
{
    return $this->_limbo(function () {
        $this->title = 'Edit Limbo Draft Entity';
        $this->view = view('your_limbo_draft_view');
        $this->vars = [
            'types' => 'My Types',
            'options' => 'My Options',
        ];
        }, function () use ($request) {
            $this->item->saveAsDraft($request->all());
            $this->redirect = redirect()->route('your_drafts_list_route');
        }, $id, $request, new YourRequest());
    }
```
> It's mandatory for `your limbo draft edit method` to only return an instance of the `_limbo()` method.   
> The entire logic should happen inside the `callback` of the `_limbo()` method.   
>   
> The `_limbo()` method requires `2 parameters` to be passed to it, under the form of `callback functions`. The `first` one is for the `GET` request (display the draft), while the second one is for the `PUT` request (saving the draft).   
>   
> Right of the bat, the `_limbo()` method will automatically set the `item` property to the `limbo draft` corresponding to the `id` provided, so you don't need to worry about that.   
>    
> **Required actions:**   
> - Set the `view` property.   
>   - This way, the controller will know which `view file` to return for your `draft edit` operation.   
>    
> - `Save` the `limbo draft`.   
>   - This is for `updating` the `limbo draft`. The `publish` itself is automatically done by the platform.   
>    
> - Set the `redirect` property.   
>   - This way, the controller will know to which `route` to redirect after the limbo draft was successfully `saved`.   
>   
> **Optional actions**   
> - Set the `title` property.   
>   - The value of the title will be appended to your page's `meta title` and it will also be available to you inside your `view file` by accessing the `$title` variable.   
> - Append to the `vars` property.   
>   - By default, the `_limbo()` method will automatically assign the `item` and `title` to your view, but you can also specify other `variables` to use inside your `view file`.   
> - Any other `custom logic` that your application might require.

**The `_revision()` method**

This method is used for showing the `revision edit view`.   
This method accepts only `GET` requests, so your route should be defined accordingly.

```php
// in your entity crud controller

/**
 * @param Revision $revision
 * @return \Illuminate\View\View
 */
public function yourDraftMethod(Revision $revision)
{
    return $this->_revision(function () use ($revision) {
        $this->item = $revision->revisionable;
        $this->item->rollbackToRevision($revision);

        $this->title = 'Edit Revision Entity';
        $this->view = view('your_revision_view');
        $this->vars = [
            'types' => 'My Types',
            'options' => 'My Options',
        ];
    }, $revision);
}
```
> It's mandatory for `your revision edit method` to only return an instance of the `_revision()` method.   
> The entire logic should happen inside the `callback` of the `_revision()` method.   
>    
> **Required actions:**   
> - Set the `item` property to the `original model`.   
>   - Once you've done that, you can access your `model` inside your `view file`, by accessing the `$item` variable.   
>    
> - `Rollback` the original model to the given `revision`.   
>   - This is done inside a `database transaction` that's later `rolled back`, but the `rollback` needs to happen in order for you to see the `revision model data` inside your `form`.
>    
> - Set the `view` property.   
>   - This way, the controller will know which `view file` to return for your `revision edit` operation.   
>    
> **Optional actions**   
> - Set the `title` property.   
>   - The value of the title will be appended to your page's `meta title` and it will also be available to you inside your `view file` by accessing the `$title` variable.   
> - Append to the `vars` property.   
>   - By default, the `_revision()` method will automatically assign the `item` and `title` to your view, but you can also specify other `variables` to use inside your `view file`.   
> - Any other `custom logic` that your application might require.

# Observations

Setting the `crud model`.   
- As specified above, a `controller` that uses the `App\Traits\CanCrud` trait, has to define the `model property`. However, if you fail in doing so, the code will throw an error, indicating this to you and also telling you what exactly needs to be done.   

Setting the `crud properties`.   
- As specified after each `method's example`, every one of the methods above require you to `instantiate certain properties` inside their callbacks. However, if you fail in doing so, the code will throw an error, indicating this to you and also telling you what exactly needs to be done.   
   
Using the `crud methods`.   
- When writing a `crud functionality`, you're `not required` to specify every one of the methods above, but just the ones you actually need.   
- If you feel like you need to know more about the methods, please feel free to check the `App/Traits/CanCrud.php` file.

The `views` and `routes`.
- This documentation only covers how to use the `App\Traits\CanCrud` trait correctly. Defining the `views` and `routes` it's up to you to develop as you wish.