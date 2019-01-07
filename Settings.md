- [Overview](#overview)   
- [Admin Entity](#admin-entity)   
- [Usage](#usage)   
  - [The Setting Model](#the-setting-model)   
  - [The Setting Helper](#the-setting-helper)   

# Overview

The `settings` entity is a core component of the platform's `config module`.   

# Admin Entity

Putting aside the settings features that the developer will use, the settings component comes bundled with a `settings entity` out of the box.   
   
You can find this section inside the `Admin -> System Settings`.
From there, you can manage your different `setting types`.

# Usage

### The Setting Model

Settings are managed by the `App\Models\Config\Setting` model.

Below you'll find some basic functionalities of the `App\Models\Config\Setting`, but you can always inspect the model's functionalities yourself, or even add some new features to it.   
   
Sort the settings by `newest`.

```php
$settings = Setting::newest()->get();
```

Sort the settings `alphabetically`.

```php
$settings = Setting::alphabetically()->get();
```

Filter settings by `key`.

```php
$settings = Setting::key('your-key`)->get();
```

Get a setting by its `key`.

```php
$setting = Setting::findByKey('your-key`);
```

### The Setting Helper

The `setting()` helper exposes 2 methods for easily fetching a setting or its value.

Get a setting by its `key`.

```php
$setting = setting()->find('your-key`);
```

Get a setting's `value`.

```php
$value = setting()->value('your-key`);
```