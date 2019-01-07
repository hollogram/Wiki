- [Overview](#overview)   
- [Usage](#usage)   
  - [The IsSortable Trait](#the-issortable-trait)   
  - [The Sort Request Object](#the-sort-request-object)   
  - [The Sorted Query Scope](#the-sorted-query-scope)   

# Overview

This feature provides a trait that allows you to easily sort Eloquent model records, based on a sort mapping.

# Usage

### The IsSortable Trait

Your models should use the `App\Traits\IsSortable` trait.

```php
use App\Traits\IsSortable;

class YourModel
{
    use IsSortable;

    ...
}
```

### The Sort Request Object

After you've used the IsSortable trait on your models, it's time to define sort request objects in order to teach the trait how it should sort your model records.   
   
A sort request object is basically a child class of the `App\Http\Sorts\Sort` class.   
   
Given that the parent class defines 2 abstract methods (`field`, `direction`), your child class should only implement these 2 methods.

```php
namespace App\Http\Sorts;

use App\Http\Sorts\Sort;

class YourSort extends Sort
{
    /**
     * Get the request field name to sort by.
     *
     * @return string
     */
    public function field()
    {
        return 'sort';
    }

    /**
     * Get the direction to sort by.
     *
     * @return array
     */
    public function direction()
    {
        return 'dir';
    }
}
```

**The `field()` method**

This method should return the name of the query string parameter that it's value represents the field name to sort by.

```
'sort' - http://example.com?sort=date
```

Inside the `App\Http\Sorts\Sort` class there is a `default field` defined by the `DEFAULT_SORT_FIELD` constant. The `App\Traits\IsSortable` will use this default value if no value is supplied in your sort request object.

**The `direction()` method**

This method should return the name of the query string parameter that it's value represents the direction to sort by.

```
'dir' - http://example.com?dir=asc
```

Inside the `App\Http\Sorts\Sort` class there is a `default direction` defined by the `DEFAULT_DIRECTION_FIELD` constant. The `App\Traits\IsSortable` will use this default value if no value is supplied in your sort request object.

All of the `direction` values supported are inside the `App\Http\Sorts\Sort` class, represented by all the constants that start with `DIRECTION_`.

### The Sorted Query Scope

The final step after you've implemented the `IsSortable` trait on your models and created your `Sort Requests`, is to actually get your `sorted results`.   
   
This can be done by using the `sorted` query scope exposed by the `IsSortable` trait.   
   
The `sorted` query scope requires 2 parameters to be passed to it:
- the current request (type of `Illuminate\Http\Request`)
- your created sort class (type of `App\Http\Sorts\Sort`)

```php
namespace App\Http\Controllers;

use App\Http\Sorts\YourSort;
use Illuminate\Http\Request;

controller YourController extends Controller
{
    public function getSortedResults(Request $request, YourSort $sort)
    {
        return YourModel::sorted($request, $sort)->get();
    }
}
```