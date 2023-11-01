<p align="center"><a href="https://spatie.be/docs/laravel-health/v1/introduction" target="_blank"><img src="https://spatie.be/docs/laravel-health/v1/images/list-web-dark.png" width="600" alt="Laravel Health"></a></p>

# Laravel Health
## Monitor the health of a Laravel application

[![Latest Version on Packagist](https://img.shields.io/packagist/v/spatie/laravel-health.svg?style=flat-square)](https://packagist.org/packages/spatie/laravel-health)
[![run-tests](https://github.com/spatie/laravel-health/actions/workflows/run-tests.yml/badge.svg)](https://github.com/spatie/laravel-health/actions/workflows/run-tests.yml)
[![Check & fix styling](https://github.com/spatie/laravel-health/actions/workflows/pint.yml/badge.svg)](https://github.com/spatie/laravel-health/actions/workflows/pint.yml)
[![Total Downloads](https://img.shields.io/packagist/dt/spatie/laravel-health.svg?style=flat-square)](https://packagist.org/packages/spatie/laravel-health)


Using this package you can monitor the health of your application by registering checks.

## Installation & setup

You can install the package via composer:
```php
composer require spatie/laravel-health
```

#### Publishing the config file
Optionally, you can publish the health config file with this command.

```php
php artisan vendor:publish --tag="health-config"
```

#### Migrating the database

```php
php artisan vendor:publish --tag="health-migrations"
php artisan migrate
```
These steps are not necessary when using the JsonFileResultStore

#### Running the checks by scheduling a command
If you want to let your application send notifications when something is wrong, you should schedule the RunHealthChecksCommand to run every minute.

```php
// in app/Console/Kernel.php
protected function schedule(Schedule $schedule)
{
    $schedule->command(\Spatie\Health\Commands\RunHealthChecksCommand::class)->everyMinute(); // hourly()
}
```

#### Try Available checks

```php
composer require doctrine/dbal
composer require spatie/cpu-load-health-check
composer require spatie/security-advisories-health-check
```

```php
use Spatie\Health\Facades\Health;
use Spatie\Health\Checks\Checks\PingCheck;
use Spatie\CpuLoadHealthCheck\CpuLoadCheck;
use Spatie\Health\Checks\Checks\CacheCheck;
use Spatie\Health\Checks\Checks\QueueCheck;
use Spatie\Health\Checks\Checks\DatabaseCheck;
use Spatie\Health\Checks\Checks\ScheduleCheck;
use Spatie\Health\Checks\Checks\EnvironmentCheck;
use Spatie\Health\Checks\Checks\MeiliSearchCheck;
use Spatie\Health\Checks\Checks\DatabaseSizeCheck;
use Spatie\Health\Checks\Checks\OptimizedAppCheck;
use Spatie\Health\Checks\Checks\UsedDiskSpaceCheck;
use Spatie\Health\Checks\Checks\DatabaseTableSizeCheck;
use Spatie\Health\Checks\Checks\DatabaseConnectionCountCheck;
use Spatie\SecurityAdvisoriesHealthCheck\SecurityAdvisoriesCheck;
```

```php
// typically, in a service provider
Health::checks([
    CacheCheck::new(),
    QueueCheck::new(),
    ScheduleCheck::new(),
    DatabaseCheck::new(),
    EnvironmentCheck::new(),
    OptimizedAppCheck::new(),
    UsedDiskSpaceCheck::new(),
    SecurityAdvisoriesCheck::new(),
    // ->ignorePackages([
        //     'spatie/laravel-backup',
        //     'spatie/laravel-medialibrary',
        // ]),

    PingCheck::new()->url(config('app.url')),
    MeiliSearchCheck::new()->url(config('app.url')),
    DatabaseConnectionCountCheck::new()->failWhenMoreConnectionsThan(100),
    CpuLoadCheck::new()->failWhenLoadIsHigherInTheLast5Minutes(2.0)->failWhenLoadIsHigherInTheLast15Minutes(1.5),

    DatabaseSizeCheck::new()->failWhenSizeAboveGb(errorThresholdGb: 5.0),

    DatabaseTableSizeCheck::new()
        ->table('users', maxSizeInMb: 1_000)
        ->table('migrations', maxSizeInMb: 2_000),

]);
```

## Documentation

All documentation is available [on our documentation site](https://spatie.be/docs/laravel-health).

## View check results

```php
https://example.com/health?fresh
```

##### View check results

```php
use Spatie\Health\Http\Controllers\HealthCheckResultsController
Route::get('health', HealthCheckResultsController::class);
```

##### As JSON

```php
use Spatie\Health\Http\Controllers\HealthCheckJsonResultsController
Route::get('health', HealthCheckJsonResultsController::class);
```

## Alternatives

If you don't like our package, do try out one of these alternatives:

- [ans-group/laravel-health-check](https://github.com/ans-group/laravel-health-check)
- [Antonioribeiro/health](https://github.com/antonioribeiro/health)
