- [Overview](#overview)   
- [Usage](#usage)   
- [How To](#how-to)  

# Overview

This feature enables you to easily add `meta tags` to your pages.   
   
**Supported Meta Tags**   
   
All normal tags:   
- `title`   
- `description`   
- `keywords`    
- etc   
   
All OG properties:   
- `title`   
- `description`   
- `type`   
- `url`   
- `image`   
- `audio`   
- `video`   
- `locale`   
- `determiner`   
- `site_name`   
   
All Twitter tags:   
- `card`   
- `site`   
- `site:id`   
- `creator`   
- `creator:id`   
- `description`   
- `title`    
- `image`   
- `image:alt`   
- `player`   
- `player:width`   
- `player:height`   
- `player:stream`   
- `app:name:iphone`   
- `app:id:iphone`   
- `app:url:iphone`   
- `app:id:ipad`   
- `app:url:ipad`   
- `app:name:googleplay`   
- `app:id:googleplay`    
- `app:url:googleplay`
 

# Usage

Use the `Meta` facade to interact with the `App\Helpers\MetaHelper` class.

`Set` a `meta tag` to be later used.

```php
Meta::set('title', 'Your Page Title');
Meta::set('description', 'Your Page Description');
Meta::set('keywords', 'your,page,keywords');
Meta::set('robots', 'index,nofollow');
```

`Get` a `meta tag` by its `name`.

```php
Meta::get('title'); // returns "Your Page Title"
Meta::get('description'); // returns "Your Page Description"
Meta::get('keywords'); // returns "your,page,keywords"
Meta::get('robots'); // returns "index,nofollow"
```

`Get` the `html code` for a `meta tag`.

```php
Meta::tag('title');
```
> Please keep in mind that the code above returns the ` full html code` for your `meta tags`, including: `meta tag`, `og property`, `twitter tag` (if they exist).   
   
> For example, the above code returns:   
> `<title>Your Page Title</title>`   
> `<meta name="title" content="Your Page Title" />`   
> `<meta property="og:title" content="Your Page Title" />`   
> `<meta name="twitter:title" content="Your Page Title" />`

Additionally you can `display multiple tags` at once.

```php
Meta::tags('title', 'image', 'description', 'keywords')
```

# How To

By default, when using the `App\Http\Controllers\Controller` as a parent for your `controllers`, you'll have instant access to a method called `setMeta()`, which you can use inside your `controllers` to set the meta for the page.

```php
$this->setMeta('title', 'Your Page Title');

// or

$this->setMeta([
    'title' => 'Your Page Title',
    'description' => 'Your Page Description',
    'keywords' => 'your,page,keywords',
]);
```