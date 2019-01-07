- [Overview](#overview)   
- [Usage](#usage)   
  - [The IsFilterable Trait](#the-isfilterable-trait)   
  - [The Filter Request Object](#the-filter-request-object)   
  - [The Filtered Query Scope](#the-filtered-query-scope)   

# Overview

This feature provides a trait that allows you to easily filter Eloquent model records, based on a filter mapping similar to Laravel's Form Request objects.

# Usage

### The IsFilterable Trait

Your models should use the `App\Traits\IsFilterable` trait.

```php
use App\Traits\IsFilterable;

class YourModel
{
    use IsFilterable;

    ...
}
```

### The Filter Request Object

After you've used the IsFilterable trait on your models, it's time to define filter request objects in order to teach the trait how it should filter your model records.   
   
A filter request object is basically a child class of the `App\Http\Filters\Filter` class.   
   
Given that the parent class defines 3 abstract methods (`morph`, `filters`, `modifiers`), your child class should only implement these 3 methods.

```php
namespace App\Http\Filters;

use App\Http\Filters\Filter;

class YourFilter extends Filter
{
    /**
     * Get the main where condition between entire request fields.
     *
     * @return string
     */
    public function morph()
    {
        return 'and';
    }

    /**
     * Get the filters that apply to the request.
     *
     * @return array
     */
    public function filters()
    {
        return [
            'search' => 'operator:like|condition:or|columns:name,content',
            'type' => 'operator:=|condition:or|columns:type',
            'start_date' => 'operator:date >=|condition:or|columns:created_at',
            'end_date' => 'operator:date <=|condition:or|columns:created_at',
        ];
    }

    /**
     * Get the modified value of a request filter field.
     *
     * @return array
     */
    public function modifiers()
    {
        return [];
    }
}
```

**The `morph()` method**   

This method establishes the conditional type (`and`, `or`) used for putting together the individual `where` conditions into the final filter query.   
   
If you want to constrain the query to match all the conditions applied to each `query string field`, you should return the `and` value inside this method.   

```php
/**
 * Get the main where condition between entire request fields.
 *
 * @return string
 */
public function morph()
{
    return 'and';
}
```
   
If you want the query to match at least one condition applied to each `query string field`, you should return the `or` value inside this method.

```php
/**
 * Get the main where condition between entire request fields.
 *
 * @return string
 */
public function morph()
{
    return 'or';
}
```

**The `filters()` method**

This method defines the actual filtering functionality for each query string field.   
It's syntax is somewhat similar to the one used inside Laravel's FormRequest objects.   
   
This method should return an array consisting of:
- array key - representing the parameter's name from the url's query string
- array value - defining the `operator`, `condition` and `columns` by which the filtering will work
   
The `operator`, `condition` and `columns` returned for each query string parameter's name, should be delimited by a `pipe |`. The `key` - `value` for each of those 3, should be delimited by a `colon :`. When defining multiple `columns` you should delimit them by a `comma ,`.

```php
/**
 * Get the filters that apply to the request.
 *
 * @return array
 */
public function filters()
{
    return [
        'search' => 'operator:like|condition:or|columns:name,content',
        'type' => 'operator:=|condition:and|columns:type',
        'size' => 'operator:between|condition:and|columns:size',
        'start_date' => 'operator:date >=|condition:and|columns:created_at',
        'end_date' => 'operator:date <=|condition:and|columns:created_at',
    ];
}
```
> With "morph" set to "and", the below query returns something like:   
>   
> `SELECT ... WHERE (search LIKE '%your_query_value%' OR search LIKE '%your_query_value%') AND (type = 'your_query_value') AND (size BETWEEN 'your_query_value' AND 'your_query_value') AND (DATE(created_at) >= your_query_value') and (DATE(created_at) <= your_query_value')`

All of the `operator` values supported are inside the `App\Http\Filters\Filter` class, represented by all the constants that start with `OPERATOR_`.   
   
Any `operator` can be defined simply by a string representing it's constant value, or by directly referencing the constant itself.

```php
'search' => 'operator:like|condition:or|columns:name,content',

// is exactly the same with

'search' => 'operator:' . static::OPERATOR_LIKE . '|condition:or|columns:name,content',
```

All of the `condition` values supported are inside the `App\Http\Filters\Filter` class, represented by all the constants that start with `CONDITION_`.   
   
Any `condition` can be defined simply by a string representing it's constant value, or by directly referencing the constant itself.

```php
'search' => 'operator:like|condition:or|columns:name,content',

// is exactly the same with

'search' => 'operator:like|condition:' . static::CONDITION_OR . '|columns:name,content',
```

You can also teach a filter to narrow it's results by a `related table`.   

```php
'search' => 'operator:like|condition:or|columns:username,person.first_name',
```
> Let's say that the filter request in which the above filter constraint exists is to applied on the `User` model (table - `users`). the `User` model defines a `has one` relation called `person()` to the `Person` model (table `people`).   
>   
> The above filter constraint instructs the query to search inside `users.username` and `people.first_name`.   
>   
> Please note that when defining related table filtering, the syntax is `relation_name.column_name` and not `table_name.column_name` as you would normally expect.

**The `modifiers()` method**

This method allows you to modify inputted query string values before they are passed to the filtering query.

```php
/**
 * Get the modified value of a request filter field.
 *
 * @return array
 */
public function modifiers()
{
    return [
        'size' => function ($modified) {
            foreach (request('size') as $size) {
                $modified[] = $size * pow(1024, 2);
            }

            return $modified;
        },
    ];
}
```
> The method above modifies the `size` query parameter, converting it from `megabytes` to `bytes`. This is useful if you want to input `megabyte` values into the query string, but your `database table` stores it's file sizes into `bytes`.
   
The `modifiers()` method should return an array of query string parameters with their data modified. The `value` for each array `key` can be a `string`, `array` or `callable`.   
   
If you choose to use a callable like in the example above, because you need to do some parsing, please note that the callable accepts one parameter that should be finally returned as a string or array, depending on your filtering condition for that query string parameter.

### The Filtered Query Scope

The final step after you've implemented the `IsFilterable` trait on your models and created your `Filter Requests`, is to actually get your `filtered results`.   
   
This can be done by using the `filtered` query scope exposed by the `IsFilterable` trait.   
   
The `filtered` query scope requires 2 parameters to be passed to it:
- the current request (type of `Illuminate\Http\Request`)
- your created filter class (type of `App\Http\Filters\Filter`)

```php
namespace App\Http\Controllers;

use App\Http\Filters\YourFilter;
use Illuminate\Http\Request;

controller YourController extends Controller
{
    public function getFilteredResults(Request $request, YourFilter $filter)
    {
        return YourModel::filtered($request, $filter)->get();
    }
}
```