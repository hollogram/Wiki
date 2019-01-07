- [Overview](#overview)   
- [Usage](#usage)   
  - [The HasMetadata Trait](#the-hasmetadata-trait)   
  - [The Migration](#the-migration)   
- [How To](#how-to)

# Overview

This feature provides a trait that will save `json encoded` data inside a table column named `metadata`.   
   
It will convert entire arrays of data corresponding to the metadata key and save them encoded.

# Usage

### The HasMetadata Trait

Your model should use the `App\Traits\HasMetadata` trait.

```php
use App\Traits\HasMetadata;

class YourModel
{
    use HasMetadata;

    ...
}
```

### The Migration

Your model referenced table should include the `metadata` column, which should be `long text`.

```php
Schema::create('table_name', function (Blueprint $table) {
    ...
    $table->longText('metadata');
    ...
});
```

# How To

Saving data inside the `metadata` column.

```php
$data = [
    'first_name' => 'John',
    'last_name' => 'Doe',

    // this is the array that will be json encoded
    // and saved inside the metadata column from the model's related database table
    'metadata' => [
        'email' => 'john.doe@example.com',
        'phone' => 'XXXXXXXXX',
        'address' => [
             'country' => 'America',
             'city' => 'New York'
        ],
    ],
];
        
Model::create($data);
```

Retrieving data from the `metadata` column.

```php
$model = Model::find(id);

$email = $model->metadata->email;
$phone = $model->metadata->phone;
$country = $model->metadata->address->country;
$city = $model->metadata->address->city;

// or if you want to avoid "Undefined property: stdClass::$property" exceptions

$email = $model->metadata('email');
$phone = $model->metadata('phone');
$country = $model->metadata('address.country');
$city = $model->metadata('address.city');
```