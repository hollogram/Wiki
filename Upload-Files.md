- [Overview](#overview)   
- [Configuration](#configuration)   
- [Admin Entity](#admin-entity)   
- [Stand-Alone Usage](#stand-alone-usage)
  - [The UploadService Component](#the-uploadservice-component)   
  - [The Upload Model](#the-upload-model)   
- [Eloquent Usage](#eloquent-usage)
  - [The Migration](#the-migration)   
  - [The HasUploads Trait](#the-hasuploads-trait)   
  - [The Uploader Helper](#the-uploader-helper)   
- [Display Usage](#display-usage)
  - [The Uploaded Helper](#the-uploaded-helper)   
  - [The Magic Method](#the-magic-method)
- [How To](#how-to)   

# Overview   

This feature lets you easily manipulate files as a stand-alone component, or in association with eloquent models.
   
# Configuration   

First and foremost, create a symlink between the `/storage/uploads` folder and `/public/uploads/` folder, so you can actually see your uploaded files in your views.   
```
php artisan uploads:link
```

You can configure your upload behavior from inside the `/config/upload.php` file.   
All options are well documented, so it should not be a problem figuring out what they mean.
   
# Admin Entity

Putting aside the file manipulations features that the developer will use, the upload component comes bundled with an `uploads entity` out of the box.   
   
You can find this section inside the `Admin -> Manage Content -> Uploads`, or by going to the `/admin/uploads` url.

The uploads entity allows the admin to manage all of the application's uploaded files.
The features it offers are:   
- upload multiple files at once to be later used inside your uploader instances   
- download existing files to your computer   
- view existing files in your browser   
- remove files both from the storage as well as the database   
- filter present files by keyword, type, size and date

Basically, this is a general place from where you can manage your uploaded files throughout the application.

# Stand-Alone Usage   

Sometimes you may want to manage files independently, not linking them to a model entity in your application.

### The UploadService Component   

The `App\Services\UploadService` class is responsible for all file manipulations.   
Use this class to manipulate files as a stand-alone component in your application.   

Upload a file.
```php
// using Laravel's Illuminate\Http\Request object
(new UploadService($request->file('your_file')))->upload();

// using the PHP's gloal variable $_FILES
(new UploadService($_FILES['your_file'])->upload();

// using an external url
(new UploadService('http://example.com/your_file_url.jpg')->upload();
``` 
   
Remove a file.
```php
$upload = Upload::find(id);

// for "unloading" operations you only need to specify the full path of an existing
// file on disk as the first argument to the UploadService contructor
(new UploadService($upload->full_path))->unload();
```

Download a file.
```php
$upload = Upload::find(id);

// for "downloading" operations you only need to specify the full path of an existing
// file on disk as the first argument to the UploadService contructor
return (new UploadService($upload->full_path))->unload();
```

Display a file.
```php
$upload = Upload::find(id);

// for "showing" operations you only need to specify the full path of an existing
// file on disk as the first argument to the UploadService contructor
return (new UploadService($upload->full_path))->show();
```

### The Upload Model

The `App\Models\Upload\Upload` is the model corresponding to the `database table` defined in `/config/upload.php`. (default `uploads`). Use this model to perform operations on the uploaded files themselves.

Check the `type` of a file.

```php
// given $upload is a loaded Upload instance

$upload->isImage(); // check if it's an image
$upload->isVideo(); // check if it's a video
$upload->isAudio(); // check if it's an audio
$upload->isFile(); // check if it's a file
```

Get the `size in MB` of a file.

```php
// given $upload is a loaded Upload instance

$upload->size_mb
```

Filter uploads by `type`.

```php
Upload::onlyTypes('image', 'video')->get();

// or

Upload::excludingTypes('audio', 'file')->get();
```

Filter uploads by `extension`.

```php
Upload::onlyExtensions('jpg', 'png')->get();

// or

Upload::excludingExtensions('mp4', 'mp3')->get();
```

Filter uploads by `size` in MB.

```php
Upload::sizeBetween(10, 100)->get();
```

For all the available methods and functionalities, feel free to check the `App\Models\Upload\Upload` model. You can also add to it's functionalities if your application requires it.

# Eloquent Usage   

Besides the stand-alone usage of the uploading component, you can also integrate the uploading functionality into your eloquent models. Basically associate files with eloquent models.   
   
The platform makes this very easy, accomplishable in few easy steps.   

### The Migration

Inside your migration, create the new upload field responsible for holding the entire file's full path.   
It's recommended to do this by using the `column()` method from the `App\Models\Upload\Upload` model.   
   
Using the `column()` method, it will create a `foreign key` between `your upload field` and the `original file from the uploads table` (or the table specified inside the /config/upload.php file).   
   
Alternatively you could create your upload field as a `string (varchar)` column.
   
```php
Schema::create('your_table', function (Blueprint $table) {
    ...
    Upload::column('image', $table);
    ...
});
```

### The HasUploads Trait

Your models should use the `App\Traits\HasUploads` trait.   
   
The `HasUploads` trait itself defines an abstract method called `getUploadConfig()` meaning that you should implement this method yourself on the models that you wish to associate files with.   
   
The `getUploadConfig()` method has the purpose of enabling you to override any `/config/upload.php` options.   
The options set in the config file are generic, but with the help of this method you can easily customize each model's uploading functionality.    
   
Please note that it is crucial for the `getUploadConfig()` to return an array structured exactly like the array from the `/config/upload.php` file.   
   
```php
use App\Traits\HasUploads;

class YourModel
{
    use HasUploads;

    /**
     * Get the specific upload config parts for this model.
     *
     * @return array
     */
    public function getUploadConfig()
    {
        return [
            'images' => [
                'styles' => [
                    'image' => [
                        'portrait' => [
                            'width' => '300',
                            'height' => '600',
                            'ratio' => true,
                        ],
                        'landscape' => [
                            'width' => '600',
                            'height' => '300',
                            'ratio' => true,
                        ],
                    ],
                ],
            ],
            'videos' => [
                'styles' => [
                    'metadata[video]' => [
                        'default' => [
                            'width' => '600',
                            'height' => '400',
                            'ratio' => true,
                        ],
                    ],
                ]
            ]
        ];
    }
}
```
> The configuration above only defines custom image and video styles for this model.   
> For a better understanding of each configuration key and how it works, please read the `/config/upload.php`.   

### The Uploader Helper

The `uploader()` helper has the purpose of enabling you to manage the `uploads library` in one simple line of code.   
   
Use this helper inside your `admin form view` to generate the code responsible for uploading and storing the files associated with your eloquent model, when adding / editing a record in the admin.   
   
The below code illustrates a basic example, using only the required methods on chaining.

```php
// the $model is an existing or new model instance 
// of the eloquent model used to associate files with
{!! uploader()->field('image')->model($model)->manager() !!}
```

Define a `label` for the field.   
When not specifically defined, the label will have the `field's` value, pretty formatted.   

```php
{!! uploader()->field('image')->label('Your Field Name')->model($model)->manager() !!}
```

Restrict the file `types` to be used.   
When not specifically defined, all file types are allowed (`image | video | audio | file`).

```php
{!! uploader()->field('image')->model($model)->types('image', 'video')->manager() !!}
```

Restrict the `extensions accepted` to be uploaded.   
When not specifically defined, all extensions are allowed.

```php
{!! uploader()->field('image')->model($model)->accept('jpg', 'png')->manager() !!}
```

`Disable` the uploader field.

```php
{!! uploader()->field('image')->model($model)->disabled()->manager() !!}
```

# Display Usage

Once you've setup your files management, be that as a stand-alone component or in association with your eloquent models, you will want to display those uploaded files in your application.   
   
To do this you have 2 options, both doing the same thing, so it's up to you to decide which one you will use.

### The Uploaded Helper

The more straight-forward option for displaying files is with the help of the `uploaded()` helper.

Display the `original` file.

```html
<!-- the $model->image should return the full path of the uploaded file -->
<!-- assuming that the $model is a loaded instance of your eloquent model -->
<!-- and the model's table field holding the path is called "image" -->
<img src="{{ uploaded($model->image)->url() }}" />
```

Display a generated `style` for the file.   
This applies only to `images` and `videos`.

```html
<!-- "your_style" should be defined in the "getUploadConfig()" method on the model -->
<!-- corresponding to the file name, which in this example is "image" -->
<img src="{{ uploaded($model->image)->url('your_style') }}" />
```

Display a generated `thumbnail` for the file.   
This applies only to `images` and `videos` and is valid only if `generate_thumbnails` config option from `/config/upload.php` is set to true. Of course, you can override this option, setting it to true from your `getUploadConfig()` method.

```html
<!-- for images, where only one thumbnail can be generated -->
<img src="{{ uploaded($model->image)->thumbnail() }}" />

<!-- for videos, where you can generate multiple thumbnails from config -->
<!-- you can also pass the thumbnail number you want to display -->
<img src="{{ uploaded($model->image)->thumbnail(3) }}" />
```

Get the `path` of an original file.

```php
uploaded($model->image)->path()
```

Get the `path` of a generated `style` for the file.   
This applies only to `images` and `videos`.

```php
uploaded($model->image)->path('your_style') />
```

Get a loaded `Upload model` based on the full path.

```php
$upload = uploaded($model->image)->load()
```

Check if a file `exists` inside the storage disk.

```php
uploaded($model->image)->exists()
```

### The Magic Method

Besides the uploaded helper, there is another method of displaying files.   
   
Please note that this only applies to files associated with eloquent models, because the `magic method` resides inside the `HasUploads` trait.
   
Given that your file's full path resides into the `image field` from the `model's database table`, you can display a file by simply accessing that field, prefixed with an `underscore _`.

```php
$model->_image // where $model is a loaded eloquent instance
```

The syntax above, simply returns an instance of the `uploaded()` helper loaded with the file's full path, so next, depending on your needs, you should call methods from the `uploaded()` helper described above: `url()`, `thumbnail()`, `path()`, `load()` or `exists()`.

# How To

Now that you have read through the documentation, let's implement something concrete.   
Below you will find a complete example of how to fully integrate the upload functionality.   
   
The example's use case contains the following necessities:   
- The `Blog` model will have 2 "upload" fields: `image` and `video`
- The uploaded image and video should be saved to database as well as storing to disk
- The `image` should have 2 custom styles: `square = 300x300px` and `portrait = 400x200px`
- The `video` should have 1 custom style: `default = 600x400px`
- There should be `5 thumbnails` for the `video`
- The `image max size` should be `20 MB`
- The `video max size` should be `100 MB`
- The `allowed extensions` for the `image` should be: `jpg` and `png`
- The `allowed extensions` for the `video` should be: `mp4`
- The `admin user` should be able to `upload both the image and the video`
- The `image` and `video` should be displayed on the `blog page`

**Let's start**   
   
Create the migration for the `blog` table.   
This migration should contain 2 fields for the `uploaded image and video files`.

```php
Schema::create('blog', function (Blueprint $table) {
    // the rest of the blog table fields

    Upload::column('image', $table);
    Upload::column('video', $table);
});
```

Make the `Blog` model use the `HasUploads` trait.   
Modify the fictive `Blog` model to include the following.

```php
use App\Traits\HasUploads;

class Blog
{
    use HasUploads;

    /**
     * Get the specific upload config parts for this model.
     *
     * @return array
     */
    public function getUploadConfig()
    {
        return [
            'images' => [
                // set the MAX SIZE to 20 MB
                // for the uploaded images
                'max_size' => 20,

                // set the ALLOWED EXTENSIONS to JPG and PNG only
                // for the uploaded images
                'allowed_extensions' => [
                    'jpg', 'png'
                ],

                'styles' => [
                    'image' => [
                        // set the SQUARE STYLE
                        // for the image
                        'square' => [
                            'width' => '300',
                            'height' => '300',
                            'ratio' => true,
                        ],
                        // set the PORTRAIT STYLE
                        // for the image
                        'portrait' => [
                            'width' => '400',
                            'height' => '200',
                            'ratio' => true,
                        ],
                    ],
                ],
            ],
            'videos' => [
                // set the MAX SIZE to 100 MB
                // for the uploaded videos
                'max_size' => 100,

                // set the ALLOWED EXTENSIONS to MP4 only
                // for the uploaded videos
                'allowed_extensions' => [
                    'mp4'
                ],

                // set the THUMBNAIL NUMBER to 5
                // for the uploaded videos
                'thumbnails_number' => 5,
                
                'styles' => [
                    'video' => [
                        // set the DEFAULT STYLE
                        // for the video
                        'default' => [
                            'width' => '600',
                            'height' => '400',
                            'ratio' => true,
                        ],
                    ],
                ]
            ]
        ];
    }
}
```
> Please note that the uploaded files are saved to database as well as stored to disk by default (specified inside the `/config/upload.php`), so there's no need to override that config setting inside our `getUploadConfig()` method.

Use the `uploader()` helper to generate 2 uploader instances for both `image` and `video`.   
The below code should be placed inside your admin blade views responsible for adding / editing blog entries, alongisde with your all other fields.

```php
<form ...>
// the rest of the fields for managing a blog

// ->field('image') -> set the blog database field to work with
// ->label('Image') -> set the field label to Image (optional)
// ->types('image') -> set the allowed file types to image only
// ->accept('jpg', 'png') -> set the allowed extensions to jpg and png only
{!! uploader()->field('image')->label('Image')->model($model)->types('image')->accept('jpg', 'png')->manager() !!}

// ->field('video') -> set the blog database field to work with
// ->label('video') -> set the field label to Video (optional)
// ->types('video') -> set the allowed file types to video only
// ->accept('mp4') -> set the allowed extensions to mp4 only
{!! uploader()->field('video')->label('Video')->model($model)->types('video')->accept('mp4')->manager() !!}

</form>
```

Finally, `display` the uploaded image and video to your front-end.   
The below code should go inside the `blade view responsible for rendering the front-end part for a blog post`.

```html
<!-- given that $blog is a loaded Blog model instance -->

<!-- display the original image for the blog post -->
<img src="{{ $blog->_image->url() }}" />

<!-- display the square style image for the blog post -->
<img src="{{ $blog->_image->url('square') }}" />

<!-- display the portrait image for the blog post -->
<img src="{{ $blog->_image->url('portrait') }}" />

<!-- display the original video for the blog post -->
<video controls>
    <source src="{{ $blog->_video->url() }}" type="video/{{ $blog->_video->getExtension() }}">
    Your browser does not support the video tag.
</video>

<!-- display the default style video for the blog post -->
<video controls>
    <source src="{{ $blog->_video->url('default') }}" type="video/{{ $blog->_video->getExtension() }}">
    Your browser does not support the video tag.
</video>
```