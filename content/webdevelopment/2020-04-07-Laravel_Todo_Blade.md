---
title: Laravel ToDo app with Blade
date: 2020-04-07T17:19:50+01:00
draft: True
categories:
  - Web Development
  - Programming
  - All
tags:
  - Laravel
---

### Create Laravel app

```php
WAUTERW-M-65P7:Laravel wauterw$ laravel new laravel-todo-blade
Crafting application...
Loading composer repositories with package information
Installing dependencies (including require-dev) from lock file
***Truncated***
Application key set successfully.
> Illuminate\Foundation\ComposerScripts::postAutoloadDump
> @php artisan package:discover --ansi
Discovered Package: facade/ignition
Discovered Package: fideloper/proxy
Discovered Package: fruitcake/laravel-cors
Discovered Package: laravel/tinker
Discovered Package: nesbot/carbon
Discovered Package: nunomaduro/collision
Package manifest generated successfully.
Application ready! Build something amazing.
```

```bash
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan --version
Laravel Framework 7.5.0
```
### Auth

```bash
WAUTERW-M-65P7:laravel-todo-blade wauterw$ composer require laravel/ui
Using version ^2.0 for laravel/ui
***Truncated***
Discovered Package: nunomaduro/collision
Package manifest generated successfully.
```

```bash
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan ui bootstrap --auth
Bootstrap scaffolding installed successfully.
Please run "npm install && npm run dev" to compile your fresh scaffolding.
Authentication scaffolding generated successfully.
```


```bash
WAUTERW-M-65P7:laravel-todo-blade wauterw$ npm install && npm run dev
***Truncated***
       Asset      Size   Chunks             Chunk Names
/css/app.css   177 KiB  /js/app  [emitted]  /js/app
  /js/app.js  1.06 MiB  /js/app  [emitted]  /js/app
```

```php
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel-todo-blade
DB_USERNAME=root
DB_PASSWORD=****
```
### Migrating the database
```
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan migrate
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table (0.48 seconds)
Migrating: 2014_10_12_100000_create_password_resets_table
Migrated:  2014_10_12_100000_create_password_resets_table (0.21 seconds)
Migrating: 2019_08_19_000000_create_failed_jobs_table
Migrated:  2019_08_19_000000_create_failed_jobs_table (0.19 seconds)
```

![Laravel](/images/2020-04-07-1.png)

### User factory and seeder

```
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan make:factory UserFactory
Factory already exists!
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan make:seeder UsersTableSeeder
Seeder created successfully.
```
So a user factory already exists. Verify the database > factories > UserFactory.php file.

```php
Insert content of UserFactory.php 
```

In the  database > seeds > DatabaseSeeder.php file we need to call the UsersTableSeeder we just created.

```php
Insert content of UsersTableSeeder.php 
```
Call the UsersTableSeeder from the `DatabaseSeeder.php` file.
```php
public function run()
    {
        $this->call(UserSeeder::class);
    }
```
Execute the `php artisan migrate:refresh --seed` command:

```bash
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan migrate:refresh --seed
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan migrate:refresh --seed
Rolling back: 2019_08_19_000000_create_failed_jobs_table
Rolled back:  2019_08_19_000000_create_failed_jobs_table (0.02 seconds)
Rolling back: 2014_10_12_100000_create_password_resets_table
Rolled back:  2014_10_12_100000_create_password_resets_table (0 seconds)
Rolling back: 2014_10_12_000000_create_users_table
Rolled back:  2014_10_12_000000_create_users_table (0 seconds)
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table (0.03 seconds)
Migrating: 2014_10_12_100000_create_password_resets_table
Migrated:  2014_10_12_100000_create_password_resets_table (0.02 seconds)
Migrating: 2019_08_19_000000_create_failed_jobs_table
Migrated:  2019_08_19_000000_create_failed_jobs_table (0.01 seconds)
Seeding: UsersTableSeeder
Seeded:  UsersTableSeeder (0.02 seconds)
Database seeding completed successfully.
```

![migration](/images/2020-04-07-2.png)
### Database actions

Create a todo migration
```
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan make:migration create_todos_table --create=todos
Created Migration: 2020_04_07_173827_create_todos_table
```

```php
Show content of Todo Migration
```

### Model
```bash
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan make:model Todo
Model created successfully.
```

```php
class Todo extends Model
{
    protected $fillable = [
        'user_id', 'name', 'description', 'completed',
    ];

    protected $casts = [
        'completed' => 'boolean',
    ];
}
```

### Create relations

```php
class User extends Authenticatable
{
    use Notifiable;

    //Other content

    public function todos()
    {
        return $this->hasMany('App\Todo');
    }
}
```

```php
class Todo extends Model
{
   public function user()
    {
        return $this->belongsTo('App\User');
    }
}
```

### Todo factory and seeder

```bash
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan make:factory TodoFactory
Factory created successfully.
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan make:seeder TodosTableSeeder
Seeder created successfully.
```


```php
Content TodoFactory
```

```php
Content TodosTableSeeder
```
Don't forget to add the seeder to the `DatabaseSeeder.php` file.

### Routes

```php
Route::get('todos', 'TodoController@index')->name('todos.index');
Route::post('todos', 'TodoController@store')->name('todos.store');
Route::get('todos/create', 'TodoController@create')->name('todos.create');
Route::put('todos/{todo}', 'TodoController@update')->name('todos.update');
Route::get('todos/{todo}', 'TodoController@show')->name('todos.show');
Route::delete('todos/{todo}', 'TodoController@destroy')->name('todos.destroy');
Route::get('todos/{todo}/edit', 'TodoController@edit')->name('todos.edit');
```

### Controller

```bash
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan make:controller TodosController --resource
Controller created successfully.
```

```php
Show content controller
```