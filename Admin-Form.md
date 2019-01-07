- [Overview](#overview)   
- [Usage](#usage)   

# Overview

This feature extends the [Laravel Collective Forms & Html](https://laravelcollective.com/docs/5.4/html) package.   
   
It adds ability to easily add `form fields` with the `form_admin()` helper or `FormAdmin` facade just like you would normally do any other field.   
   
The difference is that this form helper is for the `admin` section of the site.   
   
When adding `admin form fields` the `admin form` helper or facade wraps them inside additional `specific admin html`.

# Usage

`Open` a form.

```php
{!! form_admin()->open(['url' => 'url', 'method' => 'POST', 'files' => true]) !!}
```

`Close` a form.

```php
{!! form_admin()->close() !!}
```

Form `model binding`.

```php
{!! form_admin()->model($model, ['url' => 'url', 'method' => 'PUT', 'files' => true]) !!}
```

Create a `submit` button.

```php
{!! form_admin()->submit('value') !!}
```

Create a `hidden` input.

```php
{!! form_admin()->hidden('name', 'value') !!}
```

Create a `text` input.

```php
{!! form_admin()->text('name', 'label') !!}
```

Create a `textarea` input.

```php
{!! form_admin()->textarea('name', 'label') !!}
```

Create a `select` input.

```php
{!! form_admin()->select('name', 'label', ['op_1' => 'Opt 1', 'op_2' => 'Opt 2']) !!}
```

Create a `password` input.

```php
{!! form_admin()->password('name', 'label') !!}
```

Create a `file` input.

```php
{!! form_admin()->file('name', 'label') !!}
```

Create a `number` input.

```php
{!! form_admin()->number('name', 'label') !!}
```

Create a `email` input.

```php
{!! form_admin()->email('name', 'label') !!}
```

Create a `phone` input.

```php
{!! form_admin()->phone('name', 'label') !!}
```

Create a `checkbox` input.

```php
{!! form_admin()->checkbox('name', 'label', $value, $checked) !!}
```

Create a `radio` input.

```php
{!! form_admin()->radio('name', 'label', $value, $checked) !!}
```

Create a `editor` input using `TinyMCE`.

```php
{!! form_admin()->editor('name', 'label') !!}
```

Create a `calendar` input using `jQuery DatePicker`.

```php
{!! form_admin()->calendar('name', 'label') !!}
```

Create a `time` input using `jQuery TimePicker`.

```php
{!! form_admin()->time('name', 'label') !!}
```

Create a `color` input using `jQuery ColorPicker`.

```php
{!! form_admin()->color('name', 'label') !!}
```