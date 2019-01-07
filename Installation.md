# Default

First and foremost, please check the [Laravel Installation Guide](https://laravel.com/docs/5.4/installation).
After you have read and understood the guide, run the following commands:

Clone the repository inside an empty directory
```
git clone git@github.com:zbiller/cms.git .
```

Clone the environment file and configure it
```
.env.example ---> .env
```

Update the Composer's dependencies
```
composer install
```

Install the NPM's dependencies
```
npm install
```

Publish the assets
```
npm run production
```

Un-track filemode in GIT
```
git config core.filemode false
```

Grant directory permissions
```
chmod -R 775 storage/
```
```
chmod -R 775 bootstrap/cache/
```

Generate an application key
```
php artisan key:generate
```

Migrate the database
```
php artisan migrate
```

Seed the database
```
php artisan db:seed
```

Symlink uploads directories
```
php artisan uploads:link
```

# Advanced

When in production, after running the above commands, it's also recommended to run the following commands in order to optimize your Laravel application:

```
composer dump-autoload
```
```
php artisan clear-compiled
```
```
php artisan config:cache
```
```
php artisan optimize --force
```