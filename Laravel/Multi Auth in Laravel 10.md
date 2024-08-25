# Laravel Admin & User Login

## Multi Auth in Laravel 10

Multi auth is the first step to create any project. You must need to know how to separate and secure authentication for both user and admin. In laravel, we can do it various way. Today we use laravel breeze authentication system for creating separate user and admin login system where user also can register but admin not. You created separate guard, model, controller, db table everything for both admin and user. So it is fully secure. Follow the steps mentioned below, so you will able to do it too. This is the easiest step by step tutorial for you to create multi authentication system using laravel 10.

[Source](https://devnotez.com/laravel-admin-user-login-multi-auth-in-laravel-10/r)


#### Step One – Create a Fresh Laravel Project

To do so, simply run the follow command

```bash
composer create-project laravel/laravel authapp
cd authapp
```

#### Step Two – Install UI Authentication System

Run the following command one by one

```bash
composer require laravel/ui
php artisan ui bootstrap --auth
npm install
npm run dev
```

Before running the php artisan migrate, go to your project folder and find .env file, there provide your database information. To create a fresh new database, use phpmyadmin dashboard/panel.

#### Now run these command

```bash
php artisan migrate
npm install
npm run dev
```

#### Step Three – Create model and migration for admin

```bash
php artisan make:model Admin -m
php artisan make:model Customer -m
php artisan make:model Employee -m
```

#### Step Four – Go to admin migration file and add this column to your table

```bash
$table->id();
$table->string('name');
$table->string('email')->unique();
$table->timestamp('email_verified_at')->nullable();
$table->string('password');
$table->rememberToken();
$table->timestamps();
```

#### Step Five – Go to Admin model file and replace the code with it

```bash
<?php

namespace App\Models;

// use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;

class Admin extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;

    protected $guard = 'admin';

    /**
     * The attributes that are mass assignable.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    /**
     * The attributes that should be hidden for serialization.
     *
     * @var array<int, string>
     */
    protected $hidden = [
        'password',
        'remember_token',
    ];

    /**
     * The attributes that should be cast.
     *
     * @var array<string, string>
     */
    protected $casts = [
        'email_verified_at' => 'datetime',
        'password' => 'hashed',
    ];
}
```


Next, we have to create this guard. To do so, go to config folder and inside that, you will find auth.php, go to that file and find out guards. There you will see, one guard already available which is web, now copy the web guard code and paste just below. Next change the name from web to admin and provider value to admins which is users in web guard. Lastly go below of that file code, you will find there providers. Already there users provider available. Copy the code and paste just below of users provider and change the name from users to admin and also change the model path to Admin model path.

```bash
'model' => App\Models\Admin::class,
```
