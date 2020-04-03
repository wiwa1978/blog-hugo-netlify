---
title: Laravel ToDo app with Blade
date: 2020-03-18:19:50+01:00
draft: true
categories:
  - Programming
  - Backend
  - Frontend
tags:
  - Laravel
---

### Create Laravel app

```php
WAUTERW-M-65P7:Laravel wauterw$ laravel new laravel-todo-blade
Crafting application...
Loading composer repositories with package information
Installing dependencies (including require-dev) from lock file
Package operations: 94 installs, 0 updates, 0 removals

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
Laravel Framework 7.1.0
```
### Auth

```bash
WAUTERW-M-65P7:laravel-todo-blade wauterw$ composer require laravel/ui
Using version ^2.0 for laravel/ui
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 1 install, 0 updates, 0 removals
  - Installing laravel/ui (v2.0.1): Loading from cache
Writing lock file
Generating optimized autoload files
> Illuminate\Foundation\ComposerScripts::postAutoloadDump
> @php artisan package:discover --ansi
Discovered Package: facade/ignition
Discovered Package: fideloper/proxy
Discovered Package: fruitcake/laravel-cors
Discovered Package: laravel/tinker
Discovered Package: laravel/ui
Discovered Package: nesbot/carbon
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
98% after emitting SizeLimitsPlugin

 DONE  Compiled successfully in 6331ms                                                                                                  11:44:50

       Asset      Size   Chunks             Chunk Names
/css/app.css   177 KiB  /js/app  [emitted]  /js/app
  /js/app.js  1.06 MiB  /js/app  [emitted]  /js/app
```

```
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan migrate
Dropped all tables successfully.
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table (0.4 seconds)
Migrating: 2014_10_12_100000_create_password_resets_table
Migrated:  2014_10_12_100000_create_password_resets_table (0.05 seconds)
Migrating: 2019_08_19_000000_create_failed_jobs_table
Migrated:  2019_08_19_000000_create_failed_jobs_table (0.06 seconds)
```
![SequelPro](/images/2020-03-18-1.png)

### User factory and seeder

```
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan make:factory UserFactory
Factory already exists!
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan make:seeder UsersTableSeeder
Seeder created successfully.
```
So a user factory already exists. Verify the database > factories > UserFactory.php file.

```php
use App\User;
use Illuminate\Database\Seeder;

class UsersTableSeeder extends Seeder
{
    public function run()
    {
        factory(App\User::class, 2)->create()->each(function ($u) {
            $u->todos()->save(factory(App\Todo::class)->make());
        });
    }
}
```

In the  database > seeds > DatabaseSeeder.php file we need to call the UsersTableSeeder we just created.
```
class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     *
     * @return void
     */
    public function run()
    {
        $this->call(UsersTableSeeder::class);
    }
}
```

### Database actions

![SequelPro](/images/2020-03-18-2.png)

Changes to .env file:
```php
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel-todo-blade
DB_USERNAME=root
DB_PASSWORD=12345678a-
```
Let's continue with the database migration
```bash
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan migrate
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table (0.06 seconds)
Migrating: 2019_08_19_000000_create_failed_jobs_table
Migrated:  2019_08_19_000000_create_failed_jobs_table (0.04 seconds)
```
Create a todo migration
```
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan make:migration create_todos_table
Created Migration: 2020_03_29_094011_create_todos_table
```






```php
WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan make:model Todo
Model created successfully.



WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan make:factory TodoFactory
Factory created successfully.


WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan make:seeder TodosTableSeeder
Seeder created successfully.


WAUTERW-M-65P7:laravel-todo-blade wauterw$ php artisan make:controller TodoController --resource
Controller created successfully.
```
