## ✅ Laravel Package Development Steps

### 1️⃣ Setup Test Laravel Project

```bash
composer create-project laravel/laravel test-project
cd test-project
```

### 2️⃣ Create Package Skeleton

* Decide package name (e.g., `vendor/package-name`)
* Initialize composer inside your package folder:

```bash
composer init
```

* Fill in:

    * Vendor name
    * Package name
    * Description
    * License
    * Minimum stability

### 3️⃣ Folder Structure

```
packages/
   vendor/
      package-name/
         src/
         composer.json
```

### 4️⃣ PSR-4 Autoloading

* Inside `packages/vendor/package-name/composer.json`:

```json
"autoload": {
    "psr-4": {
        "Vendor\\PackageName\\": "src/"
    }
}
```

* Run:

```bash
composer dump-autoload
```

### 5️⃣ Register Package in Test Laravel App

* Add to root `composer.json`:

```json
"repositories": [
    {
        "type": "path",
        "url": "packages/vendor/package-name"
    }
]
```

```json
"repositories": {
    "dev-package" : {
        "type" : "vcs",
        "url": "https://github.com/devrabiul/laravel-toaster-magic",
        "options": {
          "symlink": true
        },
        "_comment": "This is the GitHub repository for the Master File Manager, can use path and ../advanced-file-manager for local",
        "__comment" : "vcs https://github.com/devrabiul/laravel-toaster-magic",
        "__comment_vsc" : "use vcs as type and url https://github.com/devrabiul/laravel-toaster-magic",
        "__comment_path" : "use path as type and url ../advanced-file-manager"
    }
}
```

* Require package locally:

```bash
composer require vendor/package-name
```

### 6️⃣ Create Service Provider

* Inside `src/`:

```php
<?php

namespace Vendor\PackageName;

use Illuminate\Support\ServiceProvider;

class PackageNameServiceProvider extends ServiceProvider
{
    public function register()
    {
        // Bind services or singletons
    }

    public function boot()
    {
        // Publish config, routes, views, migrations
    }
}
```

* Register provider in `composer.json` (package):

```json
"extra": {
    "laravel": {
        "providers": [
            "Vendor\\PackageName\\PackageNameServiceProvider"
        ]
    }
}
```

### 7️⃣ Add Core Features

* **Config file:** publishable via `php artisan vendor:publish`
* **Routes:** `routes/web.php`
* **Controllers:** inside `src/Http/Controllers`
* **Views:** inside `resources/views`
* **Migrations:** inside `database/migrations`

### 8️⃣ Version Control

* Initialize Git inside package folder:

```bash
git init
git add .
git commit -m "Initial package structure"
```

### 9️⃣ Testing Package

* Create Feature/Unit tests inside `tests/`
* Run in Laravel app:

```bash
php artisan test
```

### 🔟 Publishing

* Push package to GitHub
* Tag versions (`git tag v1.0.0`)
* Update `composer.json` in Laravel app to pull from Git repo or submit to Packagist.
