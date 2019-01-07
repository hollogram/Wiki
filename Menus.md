- [Overview](#overview)   
- [Admin Entity](#admin-entity)   
- [Usage](#usage)   
  - [The Menu Model](#the-menu-model)   
  - [The Menu View Composer](#the-menu-view-composer)   
- [How To](#how-to)   
  - [Render Menus](#render-menus)   
  - [Create New Menu Location](#create-new-menu-location)   
  - [Link Entity To Menu](#link-entity-to-menu)   
  - [Add Items To Admin Menu](#add-items-to-admin-menu)

# Overview

The `menus` entity is a core component of the platform's `cms module`, representing the `front-end menu items` divided into `menu locations`.

# Admin Entity

Putting aside the menu features that the developer will use, the block component comes bundled with a `menus entity` out of the box.   
   
You can find this section inside the `Admin -> Content Management -> Menus`, or by going to the `/admin/menus` url.   
From there, you can `add`, `edit`, `delete` menus for certain `locations`.  

# Usage

### The Menu Model

Menus are managed by the `App\Models\Cms\Menu` model.

Below you'll find some basic functionalities of the `App\Models\Cms\Menu`, but you can always inspect the model's functionalities yourself, or even add some new features to it.   

Fetch the `model instance` that a given `menu` belongs to.

```php
$menu = Menu::find($id);

$block->menuable;
```

Get the `url` of the menu.   
You can do this by using the `getUrlAttribute()` method from the model.

```php
$menu = Menu::find($id);

$menu->url; // returns the url as a string
```

Sort the menus by `newest`.

```php
$menus = Menu::newest()->get();
```

Sort the menus `alphabetically`.

```php
$menus = Menu::alphabetically()->get();
```

Fetch only `active` menus.

```php
$menus = Menu::active()->get();
```

Fetch only `inactive` menus.

```php
$menus = Menu::inactive()->get();
```

Filter menus by `parent`.

```php
$menus = Menu::whereParent($parentId)->get();
```

Filter menus by `location`.

```php
$menus = Menu::whereLocation('your-location')->get();
```

### The Menu View Composer

The `App\Http\Composers\MenuComposer` is registered by default inside the `App\Providers\ComposerServiceProvider` class.   
From here you can share `data` about `your menus` with `your blade views`.   
   
The `front()` method from the `menu composer` is for defining `front-end menus`.   
The `admin()` method from the `menu composer` is for defining `admin menus`.   

# How To

### Render Menus

After you've defined your `menu locations` and have inserted `menu items` from the `admin panel`, you'll want to display those menu items inside your `front-end`.

Inside a `blade view`, wherever you see fit, you can display the menu using the `menu()` helper.

```php
menu()->get('top')
```

> The `get()` method requires only one parameter which can be any `LOCATION constant` defined inside the `App\Models\Cms\Menu` model.   
>   
> Please note that the `get()` method will only return the `active menu roots` in the order defined by the tree in the admin panel.

The following is a `full example` of implementing a menu with `2 levels`:

```html
<ul class="navbar-nav navbar-nav-right">
    @foreach(menu()->get('top') as $index => $menu)
    <li class="nav-item">
        <a class="nav-item" href="{{ $menu->url }}" {{ $menu->metadata['new_window'] == 1 ? 'target="_blank"' : '' }}>
            {{ $menu->name }}
        </a>

        @if($menu->children)
            <ul class="sub-navbar-nav">
            @foreach($menu->children as $child)
                <li class="sub-nav-item">
                    <a class="nav-item-child" href="{{ $$child->url }}" {{ $child->metadata['new_window'] == 1 ? 'target="_blank"' : '' }}>
                        {{ $child->name }}
                    </a>
                </li>
            @endforeach
            </ul>
        @endif
    </li>
    @endforeach
</ul>
```

Alternatively, if your application doesn't use the `Menus Admin Entity` to fetch it's menu items, you can define your menu items from inside the `front()` method defined on the `App\Http\Composers\MenuComposer` class.

### Create New Menu Location

The `Menus Entity` already supports 2 locations (`top` and `footer`).   
Creating new menu locations is encouraged if you have more than just 2 places in the front-end where menu items should be displayed.

Inside the `App\Models\Cms\Menu` model define a new `LOCATION` constant.

```php
/**
 * The constants defining the menu locations.
 *
 * @const
 */
...
const LOCATION_YOUR_LOCATION = 'your-location';
```

Inside the `App\Models\Cms\Menu` model append your defined `LOCATION` constant to the `$locations` property.

```php
/**
 * The property defining the menu locations.
 *
 * @var array
 */
public static $locations = [
    ...
    self::LOCATION_YOUR_LOCATION => 'Your Location',
];
```

At this point you should be able to manage `your menu location` from inside the `admin`.

### Link Entity To Menu

Creating a `new menu type` for a `custom entity` defined by you inside your application is encouraged if you want to `link menu items` directly to a `specific record url of that entity`. _(exactly how pages can be directly linked to a menu item)_

Inside the `App\Models\Cms\Menu` model define a new `TYPE` constant.

```php
/**
 * The constants defining the entity types.
 *
 * @const
 */
...
const TYPE_YOUR_ENTITY = 'your-entity';
```

Inside the `App\Models\Cms\Menu` model append your defined entity `TYPE` constant to the `$types` property.

```php
/**
 * The property defining the entity types.
 *
 * @var array
 */
public static $types = [
    ...
    self::TYPE_NEW_ENTITY => 'New Entity',
];
```

Inside the `App\Models\Cms\Menu` model create a `new key` for your entity `TYPE` inside the `$map` property.

```php
/**
 * The options available for each menu type.
 *
 * @var array
 */
public static $map = [
    'types' => [
     	...
        self::TYPE_YOUR_ENTITY => YourModel::class,
    ],
];
```

At this point you should be able to manage `your new menu type` from inside the `admin`.

### Add Items To Admin Menu

You can add additional menu items to the `admin menu` by using the `menu()` helper inside the `App\Http\Composers\MenuComposer` class, specifically inside the `admin()` method.   
   
Follow the present implementation's example for the current admin menu items that are listed inside the admin.