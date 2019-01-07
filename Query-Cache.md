- [Overview](#overview)   
- [Configuration](#configuration)
- [Usage](#usage)   
  - [The IsCacheable Trait](#the-iscacheable-trait)   
  - [The Query Builder](#the-query-builder)   
  - [The CacheService Component](#the-cacheservice-component)   

# Overview

This functionality enables you to easily cache queries out of the box, with no extra development.   
   
There are 2 query caching functionalities that can be applied:
- `basic` - cache only duplicate queries, for the current request, using the `array` cache driver
- `advanced` - cache all queries, forever, using the `redis` cache driver

The queries are cached using specific cache keys, composed from the `query sql` itself and its `query bindings`. This way, the uniqueness of the cached query is ensured.

# Configuration

All query cache configuration options can be found inside the `/config/cache.php` file, at the `query` section.   
   
**Basic Query Cache**   
   
You can `enable` or `disable` the duplicate query caching for the application, by setting the `CACHE_DUPLICATE_QUERIES` configuration option to `true` or `false` from inside your `.env` file.   
   
**Advanced Query Cache**   
   
You can `enable` or `disable` the full and forever query caching for the application, by setting the `CACHE_QUERIES` configuration option to `true` or `false` from inside your `.env` file.   
   
Please note that in order for the `advanced cache` to work, you need to [setup Redis](https://laravel.com/docs/master/redis).   
   
> Please note that none of the cache functionalities will run when the application's environment is set to `development`, regardless the fact that you enabled the cache.   
>   
> This gives the developer the possibility to debug its code more freely.

# Usage

### The IsCacheable Trait

Your models should use the `App\Traits\IsCacheable` trait.   
By default, all models use this trait, but don't forget to add it to new ones that you might create.   
   
When a model instance is modified in some way, the entire query cache associated to this model cache key, will be flushed. This way, we ensure that no outdated content will be shown, due to the presence of a query containing old data.

```php
use App\Traits\IsCacheable;

class YourModel
{
    use IsCacheable;

    ...
}
```

### The Query Builder

The `App\Traits\IsCacheable` trait overrides the `newBaseQueryBuilder()` method from the `Illuminate\Database\Eloquent\Model`, making it return a custom builder `App\Database\Builder`, if any query caching should take place.   
   
The `App\Database\Builder` extends the `Illuminate\Database\Query\Builder` overriding its `runSelect()` method, enabling the builder to cache its queries on runtime.   
   
The `App\Database\Builder` also overrides the `delete()` and `truncate()` methods, adding extra functionality for removing queries from cache.

### The CacheService Component

This is the class responsible for handling the cache operations.   
   
Feel free to check its methods. Some of them are called inside the `App\Traits\IsCacheable` trait, others are called inside the `App\Database\Builder` class.

Verify if the `cache should run`.

```php
CacheService::shouldCache();
```

Verify if the `query cache should run`.

```php
CacheService::shouldCacheQueries();
```

Verify if the `duplicate query cache should run`.

```php
CacheService::shouldCacheDuplicateQueries();
```

`Flush` all query cache.

```php
CacheService::flushAllQueryCache();
```

`Flush` the cache `only for a model`, using its `cache key`.

```php
CacheService::clearQueryCache(YourModel::find($id));
```

Manually `disable query cache` for the current request.

```php
CacheService::disableQueryCache();
```

Verify if `query cache is disabled`

```php
CacheService::canCacheQueries();
```