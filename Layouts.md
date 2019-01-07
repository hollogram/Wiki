- [Overview](#overview)   
- [Admin Entity](#admin-entity)   
- [Usage](#usage)   
  - [The Layout Model](#the-block-model)   
- [How To](#how-to)   
  - [Create New Layout](#create-new-layout)   

# Overview

The `layouts` entity is a core component of the platform's `cms module`.   
   
This component is tightly coupled with the `pages` entity, but if you feel the need of having `layouts` on other `custom entities`, you can implement it yourself, just like how the `pages` entity implements them.

# Admin Entity

Putting aside the layout features that the developer will use, the layout component comes bundled with a `layouts entity` out of the box.   
   
You can find this section inside the `Admin -> Content Management -> Layouts`, or by going to the `/admin/layouts` url.   
From there, you can `add`, `edit`, `delete` layouts.

# Usage

### The Layout Model

Layouts are managed by the `App\Models\Cms\Layout` model.

Below you'll find some basic functionalities of the `App\Models\Cms\Layout`, but you can always inspect the model's functionalities yourself, or even add some new features to it.   

Get the `pages` belonging to a layout.   
By default, the `pages` entity supports `layout assignment`, through a `one to many` relation called `pages()`.

```php
$layout = Layout::find($id);

$layout->pages;
```

Get the `blade attribute`.   
This represents the layout `file` for the given layout `type` defined inside the `$map` property.   
The `getBladeAttribute()` method will return the layout `file` formatted for usage inside `blade directives`.

```php
$layout = Layout::find($id);

$layout->blade;
```

Sort the layouts by `newest`.

```php
$layouts = Layout::newest()->get();
```

Sort the layouts `alphabetically`.

```php
$layouts = Layout::alphabetically()->get();
```

Filter the layouts by `identifier`.

```php
$layouts = Layout::whereIdentifier('your-identifier')->get();
```

Filter the layouts by `type`.

```php
$layouts = Layout::whereType(Layout::TYPE_DEFAULT)->get();
```

Filter the layouts by `multiple types`.

```php
$layouts = Layout::whereTypeIn([Layout::TYPE_DEFAULT, Layout::TYPE_HOME])->get();
```

# How To

### Create New Layout

More often than not, your front-end will have `different styled pages`.    
This fact means that you will need `multiple layouts` inside your `cms section`.   
   
Inside the `App\Models\Cms\Layout` model define a new `TYPE` constant.

```php
/**
 * The constants defining the layout type.
 *
 * @const
 */
 ...
 const TYPE_YOUR_TYPE = 100;
```

Inside the `App\Models\Cms\Layout` model append your defined `TYPE` constant to the `$types` property.

```php
/**
 * The property defining the page types.
 *
 * @var array
 */
public static $types = [
    ...
    self::TYPE_YOUR_TYPE => 'Your Type',
];
```

Inside the `App\Models\Cms\Layout` model create a `new key` for your `TYPE` inside the `$map` property.

```php
/**
 * The options available for each layout type.
 *
 * --- label
 * The layout name displayed throughout the application.
 *
 * --- block_locations
 * The available block locations for the layout.
 * The available block locations for pages inheriting the layout.
 *
 * @var array
 */
public static $map = [
    ...

    self::TYPE_YOUR_TYPE => [
        'file' => 'default/your_layout_file.blade.php',
        'block_locations' => [
            'header', 'content', 'footer'
        ],
    ],
];
```

Finally create your `layout file` defined in the `file` key from the `$map` property inside the `/resources/layouts` directory.   
   
Following the example above, you should create a `blade file` called `your_layout_file.blade.php` inside `/resources/layouts/default` directory.