# PhpStorm autocomplete support
* composer install **barryvdh/laravel-ide-helper** (if not already installed)
* add `'Barryvdh\LaravelIdeHelper\IdeHelperServiceProvider::class'` in config/app.php under `providers` (Package Service Providers)
* run this to generate helper file: `php artisan ide-helper:generate`