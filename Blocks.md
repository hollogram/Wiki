- [Overview](#overview)   
- [Admin Entity](#admin-entity)   
- [Usage](#usage)   
  - [The Block Model](#the-block-model)   
  - [The HasBlocks Trait](#the-hasblocks-trait)   
  - [The BlockOptions Class](#the-blockoptions-class)   
- [How To](#how-to)   
  - [Create New Block](#create-new-block)   
  - [Enable Blocks On Entity](#enable-blocks-on-entity)   
  - [Multiple Items Blocks](#multiple-items-blocks)

# Overview

The `blocks` entity is a core component of the platform's `cms module`.   
   
This component also provides a trait that allows you to easily enable other entities inside your application to support block assignment.

# Admin Entity

Putting aside the block features that the developer will use, the block component comes bundled with a `blocks entity` out of the box.   
   
You can find this section inside the `Admin -> Content Management -> Blocks`, or by going to the `/admin/blocks` url.   
From there, you can `add`, `edit`, `delete` blocks.

# Usage

### The Block Model

Blocks are managed by the `App\Models\Cms\Block` model.

Below you'll find some basic functionalities of the `App\Models\Cms\Block`, but you can always inspect the model's functionalities yourself, or even add some new features to it.   

Fetch all `model records` for a block.

```php
$block = Block::find($id);

$block->blockables(YourModel::class)->get();
```

Sort the blocks by `newest`.

```php
$blocks = Block::newest()->get();
```

Sort the blocks `alphabetically`.

```php
$blocks = Block::alphabetically()->get();
```

Get all block `locations`.

```php
$locations = Block::getLocations();
```

Get all block `types`.

```php
$types = Block::getTypes();
```

### The HasBlocks Trait

Your models should use the `App\Traits\HasBlocks` trait and implement the `getBlockOptions()` method.

```php
use App\Traits\HasBlocks;
use App\Options\BlockOptions;

class YourModel
{
    use HasBlocks;

    /**
     * Get the options for the HasBlocks trait.
     *
     * @return BlockOptions
     */
    public static function getBlockOptions()
    {
        return BlockOptions::instance();
    }
}
```

Please note that defining the `getBlockOptions()` method is mandatory.   
If you fail in doing so, the code will throw an error indicating this to you.   
    
The `getBlockOptions()` should always return an `App\Options\BlockOptions` instance.   
   
You can view all options and their methods of implementation inside the `App/Options/BlockOptions.php` file.   
   
**Trait Methods**   
   
Get the `blocks` for a model instance.   
The `App\Traits\HasBlocks` trait defines a `morph to many` relation to the `App\Models\Cms\Block` model, called `blocks`.

```php
$model->blocks // returns all blocks assigned to the model instance
```

Manually specify `not to save` the blocks when `saving the model`.   
The `assigned blocks` will remain intact. They will not be deleted.

```php
$model = YourModel::find($id);
$model->name = 'Name modified';

$model->doNotSaveBlocks()->save();
```

Get a model instance's `assigned blocks` inside a `location`.

```php
$model = YourModel::find($id);

$model->getBlocksInLocation('header');
```

Get `all existing blocks` from the database (`assigned` or `unassigned` to the model) that `can` belong inside a given `location`.

```php
$model = YourModel::find($id);

$model->getBlocksOfLocation('header');
```

Get a model instance's `inherited blocks` for a `location`.

```php
$model = YourModel::find($id);

$model->getInheritedBlocks('header');
```

Get all available `block locations` for the given model.

```php
$model = YourModel::find($id);

$model->getBlockLocations();
```

Get a list with `all` of the `block locations` currently `assigned` in database for this model instance.

```php
$model = YourModel::find($id);

$model->getExistingBlockLocations();
```

`Assign` a block to a model instance, inside a specific `location`.

```php
$model = YourModel::find($id);
$block = Block::find($id);

$model->assignBlock($block, 'header');
```

`Un-assign` a block from a model instance, from a specific `location`.

```php
$model = YourModel::find($id);
$block = Block::find($id);

// you need to pass the "id" of the record from the "pivot table"
$model->unassignBlock($block, 'header', $pivotId);
```

### The BlockOptions Class

> With the help of the `getBlockOptions()` method defined on your given models, you can customize the trait's behavior.   

Set the `locations` available to `assign blocks` in.

```php
/**
 * Get the options for the HasBlocks trait.
 *
 * @return BlockOptions
 */
public static function getBlockOptions()
{
    return BlockOptions::instance()
        ->setLocations(['header', 'sidebar', 'footer']);
}
```

Specify an entity to `inherit` blocks from.   
Your inheritor entity should also use the `App\Traits\HasBlocks` trait.

```php
/**
 * Get the options for the HasBlocks trait.
 *
 * @return BlockOptions
 */
public static function getBlockOptions()
{
    return BlockOptions::instance()
        ->inheritFrom(YourModel::find($id));
}
```

# How To

### Create New Block

If your application has `editable pieces of content` that are just too simple to be full-fledged entities, is recommended that you create a `block` for them.   
   
Inside your `console` run the following `artisan` command.

```
php artisan make:block YourBlock
```
> Please note that `YourBlock` is the `block's name`, which is the the only and required parameter for the command. Block names should not contain spaces under any circumstances and it's also recommended that they should only contain letters.   
>   
> The above command will scaffold a block's `view composer` file, as well as a `front and admin blade view` files. The created files will be inside the `App/Blocks/YourBlock/` directory.

In the `App\Models\Cms\Block` model append your `new block` to the `$blocks` property.

```php
/**
 * List with all blocks.
 *
 * --- Label:
 * The pretty formatted block type name.
 *
 * --- Composer Class:
 * The full namespace to the block's view composer.
 *
 * --- Views Path:
 * The full path to the block's views directory.
 *
 * --- Preview Image:
 * The name of the image used as block type preview in admin.
 * All of the preview images should be placed inside /resources/assets/img/admin/blocks/ directory.
 * Running "gulp" is required for migrating new images to the /public directory.
 *
 * @var array
 */
public static $blocks = [
    ...

    'YourBlock' => [
        'label' => 'Your Block',
        'composer_class' => 'App\Blocks\YourBlock\Composer',
        'views_path' => 'app/Blocks/YourBlock/Views',
        'preview_image' => 'your_block.jpg',
    ],
];
```

In `App/Blocks/YourBlock/Views/admin.blade.php` setup your block's specific `form fields` that the admin will manage when `adding` or `editing` a block of this type.

```html
{!! form_admin()->text('metadata[title]', 'Title') !!}
{!! uploader()->field('metadata[image]')->label('Image')->model($item)->types('image')->manager() !!}
{!! form_admin()->select('metadata[active]', 'Active', ['0' => 'No', '1' => 'Yes']) !!}
{!! form_admin()->calendar('metadata[date]', 'Date') !!}
{!! form_admin()->editor('metadata[content]', 'Content') !!}
```

In `App/Blocks/YourBlock/Views/front.blade.php` write the specific HTML for this block, injecting it's data properly.

```html
@if($block->anchor)
    <a id="{{ $block->anchor }}"></a>
@endif

@if(isset($data->title))
    <h1>{{ $data->title }}</h1>
@endif
@if(isset($data->image))
    <img src="{{ uploaded($data->image)->url() }}" />
@endif
```

### Enable Blocks On Entity

More often than not, you will build some custom entities in your project that will be better off if they can support blocks assignment. _(just like Pages)_

To fully achieve this, follow the steps below.   
   
The first steps are described in the [Usage Section](#usage) and those are:   
- use the `HasBlocks` trait on your models   
- configure the `HasBlocks` trait with the help of the `getBlockOptions()` method   
   
After that, it's time to enable `blocks management` on your `custom entity` inside the `admin`.   

> Please note that the full admin implementation below assumes the following:   
> - you're using the `App\Traits\CanCrud` trait on your `admin crud controllers`   
> - you're using the same `blade views structure` for `crud` already defined in the other existing entities   
>   
> If you want, you can use your own methods, but be aware that you might have to write additional logic, different from what's described below.   
   
In your entity's `_tabs.blade.php` file, add the `blocks tab` to be able to navigate to the `blocks list` when `editing` a model record.

```php
@if($item->exists)
    {!! block()->tab($item) !!}
@endif
```

> Please note that the `blocks tab` should be shown only when:   
> - the model exists   
   
In your entity's `_form.blade.php` file, right before closing the form, add the `blocks container`, responsible for displaying the `blocks list` for a model instance.

```php
@if($item->exists)
    {!! block()->container($item) !!}

    // if your model uses "drafts" or "revisions" specify it like this
    {!! block()->container($item, isset($on_draft) ? $draft : null, isset($on_revision) ? $revision : null, isset($on_revision) ? true : false) !!}
@endif
```

> Please note that the `blocks container` should be shown only when:   
> - the model exists   

### Multiple Items Blocks

Sometimes, there are cases in which a block should support `multiple (undefined number)` chunks of HTML in itself.   
_Example: 3 items on row._   
   
In this case, the `block` should be setup to support these `multiple items`.   
Please note that a block can also have `normal data` alongside these `multiple items`.   
   
In `App/Blocks/YourBlock/Views/admin.blade.php` enable support for `adding`, `ordering`, `removing` multiple items.

```html
<a id="block-add-item" class="btn dark-blue full centered no-margin-left no-margin-right no-margin-bottom">
    <i class="fa fa-plus"></i>&nbsp; Add new item
</a>
<div id="block-items-container">
    @if($item->exists && isset($item->metadata->items))
        @foreach($item->metadata->items as $index => $_item)
            <div class="block-item" data-index="{{ $index }}">
                {!! block()->buttons() !!}

                {!! form_admin()->text('metadata[items][' . $index . '][title]', 'Title') !!}
                {!! form_admin()->text('metadata[items][' . $index . '][subtitle]', 'Subtitle') !!}
                {!! form_admin()->select('metadata[items][' . $index . '][active]', 'Active', ['0' => 'No', '1' => 'Yes']) !!}
                {!! form_admin()->calendar('metadata[items][' . $index . '][date]', 'Date') !!}
                {!! form_admin()->time('metadata[items][' . $index . '][time]', 'Time') !!}
                {!! form_admin()->color('metadata[items][' . $index . '][color]', 'Color') !!}
                {!! form_admin()->editor('metadata[items][' . $index . '][content]', 'Content') !!}
                {!! uploader()->field('metadata[items][' . $index . '][image]')->label('Image')->model($item)->types('image')->manager() !!}
            </div>
        @endforeach
    @endif
</div>
<script type="x-template" id="block-items-template">
    <div class="block-item" data-index="#index">
        {!! block()->buttons() !!}

        {!! form_admin()->text('metadata[items][#index][title]', 'Title', '#title#') !!}
        {!! form_admin()->text('metadata[items][#index][subtitle]', 'Subtitle', '#subtitle#') !!}
        {!! form_admin()->select('metadata[items][#index][active]', 'Active', ['0' => 'No', '1' => 'Yes'], '#active#') !!}
        {!! form_admin()->calendar('metadata[items][#index][date]', 'Date', '#date#') !!}
        {!! form_admin()->time('metadata[items][#index][time]', 'Time', '#time#') !!}
        {!! form_admin()->color('metadata[items][#index][color]', 'Color', '#color#') !!}
        {!! form_admin()->editor('metadata[items][#index][content]', 'Content', '#content#') !!}
        {!! uploader()->field('metadata[items][#index][image]')->label('Image')->model($item)->types('image')->manager() !!}
    </div>
</script>
```

> **Let's explain the above code**   
>   
> - The `a#block-add-item` represents the `button for adding` extra items to the block's list of items and should remain as in this example, because the CSS for the Admin Panel includes styles for this also   
> - The `div#block-items-container` should contain the already `existing block items`. This is only used when `editing` a block.   
> - The `script#block-items-template` represents the template that the already existing `JavaScript code` will use when `creating` new block items.   
> - The `block()->buttons()` is a helper method that renders the buttons necessary for interacting with the block items: `move up`, `move down` and `remove`   
>      
> **Changes you should make**   
> 
> - You should only modify the `form fields` present both in `div#block-items-container` and `script#block-items-template`   
>   - For the `div#block-items-container` don't forget to specify the `$index` variable for each field name, like in the above example.   
> - For the `script#block-items-template` is a more JavaScript-ish syntax   
>   - The `index` it should always be like in the example above: `#index` _(That string will be replaced by the JavaScript code with the next numeric index in set)_   
>   - For the `value` of each `form_admin() field` you should specify a string containing the last key of the field's name, with a `#` at the start and end: `#title#` --- if field name = metadata[items][#index][`title`] _(That string will be replaced with null when cloning an existing block item. Without this, the values will be duplicated also)_