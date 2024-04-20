
### Installation

Install Laravel Module nwidart with...
To install through Composer, by run the following command:

```bash
composer require nwidart/laravel-modules
```

The package will automatically register a service provider and alias.

Optionally, publish the package's configuration file by running:

```bash
php artisan vendor:publish --provider="Nwidart\Modules\LaravelModulesServiceProvider"
```

### Creating a module

Creating a module is simple and straightforward. Run the following command to create a module.

```bash
php artisan module:make <module-name>
```

### Some Commands
```bash
php artisan module:make-factory BlogPostsFactory BlogStack
```

### Autoloading
By default the module classes are not loaded automatically. You can autoload your modules using psr-4. For example :

```bash
{
  "autoload": {
    "psr-4": {
      "App\\": "app/",
      "Modules\\": "Modules/"
    }
  }
}
```

[Link](https://nwidart.com/laravel-modules/v6/basic-usage/creating-a-module)

[New Link](https://laravelmodules.com/docs/v10/introduction)
