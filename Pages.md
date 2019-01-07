- [Overview](#overview)   
- [Admin Entity](#admin-entity)   
- [Usage](#usage)   
  - [The Page Model](#the-page-model)   
  - [The Page Helper](#the-page-helper)   
- [How To](#how-to)   
  - [Create New Page Type](#create-new-page-type)   
  - [Setup Page Type Routing](#setup-page-type-routing)

# Overview

The `pages` entity is a core component of the platform's `cms module`.   
   
The `pages` entity supports:    
- `blocks`, `url`, `tree`, `drafts`, `revisions`, `duplicates`, `activity log`, `metadata`, `cache`, `filter`, `sort`, `soft delete`.   
   
Because of using this many functionalities, this can be a `starting point` for you when developing these features for your `custom entities`.

# Admin Entity

Putting aside the page features that the developer will use, the page component comes bundled with a `pages entity` out of the box.   
   
You can find this section inside the `Admin -> Content Management -> Pages`, or by going to the `/admin/pages` url.   
From there, you can `add`, `edit`, `delete` pages.  

# Usage

### The Page Model

Pages are managed by the `App\Models\Cms\Page` model.

Below you'll find some basic functionalities of the `App\Models\Cms\Page`, but you can always inspect the model's functionalities yourself, or even add some new features to it.   
   
Get the `layout` of a page.   
The `Page` model defines a `one to one` relation to the `Layout` model.

```php
$page->layout;
```
Get the `route action` defined in the corresponding `key` of your `page type` inside the `$map` property from the model.

```php
$page->route_action;
```

Get the `route view` defined in the corresponding `key` of your `page type` inside the `$map` property from the model.

```php
$page->route_view;
```

Sort the pages by `newest`.

```php
$pages = Page::newest()->get();
```

Sort the pages `alphabetically`.

```php
$pages = Page::alphabetically()->get();
```

Fetch only `active` pages.

```php
$pages = Page::active()->get();
```

Fetch only `inactive` pages.

```php
$pages = Page::inactive()->get();
```

Filter pages by `parent`.

```php
$pages = Page::whereParent($parentId)->get();
```

Filter pages by `identifier`.

```php
$pages = Page::whereIdentifier('your-identifier')->get();
```

### The Page Helper

If you want to use page logic inside your `blade views`, you can use the `page()` helper.   
This helper exposes 5 basic methods from where you can extend your logic.

```php
// returns the loaded Page Model instance by it's identifier
// it also searches for that identifier in "deleted pages".
page()->find('your-identifier');

// returns all pages
// it also returns "deleted pages".
page()->all();

// returns only absolute parent pages
// pages that do not have a parent page
// it also returns "deleted pages".
page()->roots();

// returns the children pages of a page
// it also returns "deleted pages"
page()->children($parent);

// get a new builder instance with global scopes applied
// from this you can extend your logic when fetching from database
page()->query();
```

# How To

### Create New Page Type

Creating new `page types` is encouraged when you will have `different pages` implementing `different layouts` or `different / additional custom logic in the front-end`.  
   
If you want to see for what defining new page types is useful, please feel free to check the `App\Http\Controllers\Front\Cms\PagesController`, more precisely the `__construct()` and `show()` methods.   
   
Basically what is happening is routing `different types of pages` to `different controller methods`, returning `different views`. You can write you `type custom logic` inside the controller `methods`.

Inside the `App\Models\Cms\Page` model define a new `TYPE` constant.

```php
/**
 * The constants defining the page type.
 *
 * @const
 */
 ...
 const TYPE_YOUR_TYPE = 100;
```

Inside the `App\Models\Cms\Page` model append your defined `TYPE` constant to the `$types` property.

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

Inside the `App\Models\Cms\Page` model create a `new key` for your `TYPE` inside the `$map` property.

```php
/**
 * The options available for each page type.
 *
 * --- action
 * The action from the controller to be used for pages on the front-end.
 * The controller is the one defined in the routeUrlTo() method from the getUrlOptions() method.
 *
 * --- view
 * The view to be used for pages on the front-end.
 *
 * --- layouts
 * The list of layout types that can be assigned to a page of that type.
 *
 * @var array
 */
public static $map = [
    ...
    self::TYPE_MY_TYPE => [
        'action' => 'yourType',
        'view' => 'your_type_view',
        'layouts' => [
            Layout::TYPE_DEFAULT,
        ],
    ],
];
```

### Setup Page Type Routing

Inside the `App\Models\Cms\Page` the `App\Traits\HasUrl` trait it's used, meaning that each page, will point on front-end to a unique url.   

That means the `page model` defines a method called `getUrlOptions()`.   
Inside this method, the `page routing` is decided by the `routeUrlTo()` method.   
This method defines the `controller` and `action` that should be triggered when a user accesses that page's url.

By default, the `App\Http\Controllers\Front\Cms\PagesController` controller and `show()` action method are defined. This means that the `main executable place` for a page on front-end is that method from that controller.  
 
If you were to inspect that `show()` method, you will find out that it also `forwards` the execution to the specific action method defined in the `$map` property from the `page model`, for that `page type`.  

In `App\Http\Controllers\Front\Cms\PagesController` create a new `action method`.

```php
/**
 * Method to execute when viewing a page of type "your type".
 *
 * @return \Illuminate\View\View
 */
public function yourType()
{
    // your custom logic goes here
    
    return view($this->page->route_view);
}
```
> Please note that the `action method's name` must be the one defined in the `$map` property -> `TYPE_YOUR_TYPE` key -> `action` key from the `page model`.

Create a `new blade view` file for your `page type` inside the `/resources/views/` directory.
> Please note that the subsequent `folder structure` is defined by your value from the `$map` property -> `TYPE_YOUR_TYPE` key -> `view` key from the `page model`.