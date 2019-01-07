- [Overview](#overview)   
- [Usage](#usage)   
  - [The Migration](#the-migration)   
  - [The IsOrderable Trait](#the-isorderable-trait)   
  - [The OrderOptions Class](#the-orderoptions-class)
- [How To](#how-to)
  - [Basic Implementation](#basic-implementation)
  - [Admin Implementation](#admin-implementation)   

# Overview

This feature provides a trait that adds ordering behavior to an Eloquent model.   

The value of the order column of a new record of a model is determined by the maximum value of the order column of all records of that model + 1.

# Usage
   
### The Migration

Your model referenced table should include the `ord` column, which is the `default column name` and it should be `integer`.

```php
Schema::create('table_name', function (Blueprint $table) {
    ...
    $table->integer('ord')->default(0);
    ...
});
```

### The IsOrderable Trait

Your model should use the `App\Traits\IsOrderable` trait and implement the `getOrderOptions()` method. 

```php
use App\Traits\IsOrderable;
use App\Options\OrderOptions;

class YourModel
{
    use IsOrderable;

    /**
     * Get the options for the IsOrderable trait.
     *
     * @return OrderOptions
     */
    public static function getOrderOptions()
    {
        return OrderOptions::instance();
    }
}
```

Please note that defining the `getOrderOptions()` method is mandatory.   
If you fail in doing so, the code will throw an error indicating this to you.   
    
The `getOrderOptions()` should always return an `App\Options\OrderOptions` instance.   
   
You can view all options and their methods of implementation inside the `App/Options/OrderOptions.php` file.

### The OrderOptions Class

> With the help of the `getOrderOptions()` method defined on your given models, you can customize the trait's behavior.   

Set another `order column` for the database table.   
Don't forget to change the migration also.

```php
public static function getOrderOptions()
{
    return OrderOptions::instance()
        ->setOrderColumn('order_column');
}
```

Disable ordering when creating a model.   
By default, when creating a new record, its order number will be `the highest existing order number + 1`. Disabling this will leave the order column to its `default value of 0`.

```php
public static function getOrderOptions()
{
    return OrderOptions::instance()
        ->doNotOrderWhenCreating();
}
```

# How To

### Basic Implementation

Set a `new order` for all the records.

```php
/**
 * the record for model id 3 will have ord value 1
 * the record for model id 1 will have ord value 2
 * the record for model id 2 will have ord value 3
 */
YourModel::setNewOrder([3,1,2]);
```

Optionally you can pass the `starting order number` as the second argument.

```php
/**
 * the record for model id 3 will have ord value 11
 * the record for model id 1 will have ord value 12
 * the record for model id 2 will have ord value 13
 */
YourModel::setNewOrder([3,1,2], 10);
```

Move a model instance `up` or `down`.

```php
$model = YourModel::find($id);

$model->moveOrderUp();
$model->moveOrderDown();
```

Move a model instance to the `first` or `last` position.
```php
$model = YourModel::find($id);

$model->moveToStart();
$model->moveToEnd();
```

`Swap` the order of two models

```php
YourModel::swapOrder($model, $anotherModel);
// where $model and $anotherModel are 2 loaded model instances
```

### Admin Implementation

After you've setup your models to support the ordering feature, you'll want to enable that on your entities, inside the admin panel.   
   
You can easily accomplish that by following the above steps.   
   
On the `entity's crud controller` add the `CanOrder` trait.   
This will define the order method responsible with the entire ordering functionality.

```php
use App\Traits\CanOrder;

class YourEntityController extends Controller
{
    use CanOrder;
    ...
{
```

In `routes/web.php` add a route for the `order()` method defined by the `CanOrder` trait.

```php
Route::patch('order', ['as' => 'admin.your_entity.order', 'uses' => 'YourEntityController@order']);
```

Inside the `view responsible with listing the entity's records in admin` add the following on the `<table>` tag.

```html
<table
    data-orderable="true"   
    data-order-url="{{ route('admin.your_entity.order') }}"   
    data-order-model="{{ \App\Models\YourModel::class }}"   
    data-order-token="{{ csrf_token() }}"
>
...
</table>
```

For the `table` described above, add the following to its `<tr>` tag inside `<thead>`.

```html
<table ...>
    <thead>
        <tr class="nodrag nodrop">
            ...
        </tr>
    </thead>
    ...
</table>
```

For the `table` described above, add the following to each of its `<tr>` tags inside `<tbody>`.

```html
<table ...>
    ...
    <tbody>
        <!-- given that $item is a loaded model instance -->
        <tr id="{{ $item->id }}">
            ...
        </tr>
    </tbody>
    ...
</table>
```

> As a general note, the interaction of ordering the records is done using `TableDND`.   
>    
> The plugin has been fully configured to work with the above setup, so there is no need for front-end changes.