---
title: laravel使用artisan
date: 2019-11-04 20:30:13
tags: artisan
categories:
  - [後端路線, framework, laravel]
---

# 介紹

artisan 是 laravel 的 cli 工具，這種框架自帶的工具，在中國那邊都稱作腳手架，整個專案製作過程都離不開它，我們會用它來建 model、建 controller
諸如:

網站進入維修

    php artisan down

本機啟動 server

    php artisan serve 類似於php -S localhost:8000

建 controller

    php artisan make:controller Admin/AdminController

要看**所有指令**

    php artisan help

目前是把常用的另列出來，之後再一一分開來筆記

# 常用指令

## 資料庫遷移

用在專案初始化時

    php artisan migrate

下這個指令之後，就可以將`database\migrations`裡面 laravel 預設的三張表新增到資料庫
遭遇狀況:

    PDOException::("SQLSTATE[42000]: Syntax error or access violation: 1071 Specified key was too long; max key length is 767 bytes")

解決辦法:(https://laravel-news.com/laravel-5-4-key-too-long-error)

```php
//app/Providers/AppServiceProvider.php
    public function boot()
    {
        Schema::defaultStringLength(191);
    }
```

或是改`config\database.php`裡面的字符集

```php
        'mysql' => [
            'charset' => 'utf8',
            'collation' => 'utf8_unicode_ci',
        ],
```

然後要刪掉 migration 檔案之前，要先做一次
php artisan migration:reset
把 DB 的 migration 資料表清空不然就會遭遇
Migration not found: 2020_03_04_064053_create_user_power_table
此時就要去資料庫排刪掉這一行來排錯
https://github.com/laravel/framework/issues/2105#issuecomment-22701759
If you deleted the file you should delete the matching row from the migrations table in your DB. Don't delete a migration without doing migrate:reset first.

## 建立資料表

將 Admin 表新增到`database\migrations`裡面

    php artisan make:migration admin

或者新增一個 model 加參數-m，推薦這個，會幫忙建 migration，並且有表格基礎欄位
或者新增一個 model 加參數-a，會幫忙建 migrantion controller factory
那麼在以前 4.X 的 laravel 版本呢，我們會有個 model 資料夾，現在沒有了，原因是因為不想混淆使用者，詳情請看[框架不應該有「MODELS」資料夾](http://blog.turn.tw/?p=1541)

    php artisan make:model Model/Admin -a

然後一樣

    php artisan migrate

就會幫忙建表進去了，已經有的表就不會重複建

剩下看文檔(https://laravel.tw/docs/5.2/migrations)

## 塞假資料

阿如果要塞假資料，記得將該 model 的假資料規則加進`database\factories\AdminFactory`，就可以用工廠批量生產資料，用來測試用

    php artisan make:factory AdminFactory

我們可以怎麼做呢?

1.  手動打指令
    使用 tinker 來打 php 程式碼，tinker 就會執行，說是一種 read-eval-print-loop
    要確定在建完 factory 之後在打開此命令行，不然會找不到

        Class 'App/Model/Admin' not found in /var/www/html/lighthouse-tutorial/vendor/laravel/framework/src/Illuminate/Database/Eloquent/FactoryBuilder.php on

        php artisan tinker 類似於 php --interactive php -a

    接著打上 php 的 code，可以操作資料庫，立即看到結果，不用在那邊**var_dump**，阿如果要塞假資料，記得將該 model 的假資料規則加進`database\factories`，之後就可以用

```php
    //看看多少使用者
    App\User::count();
    //新增使用者
    factory('App\User',10)->create();
    //打印

```

2.  用 seeder
    seeder 可以個別或引入工廠的方法來塞假資料，跟上面不同的是，以個是重新手打，一個記在程式碼後執行一次執行，有什麼好處呢?因為資料庫總是有一些要初始化的資料，例如標籤阿、管理員等

        php artisan make:seeder AdminTableSeeder

個別塞

```php
    public function run()
    {
        DB::table('admin')->insert([
            'name' => str_random(10),
            'email' => str_random(10).'@gmail.com',
            'password' => bcrypt('secret'),
        ]);
    }
```

工廠批量塞，這邊要注意文章和使用者是有關連性的，一定要有使用者才會有文章

```php
    //AdminTableSeeder
    public function run()
    {
        factory(App\User::class, 10)->create()->each(function($u) {
            $u->posts()->save(factory(App\Post::class)->make());
        });
    }
```

或者

```php
    //PostFactory
    $factory->define(Post::class, function (Faker $faker) {
        return [
            'user_id' => factory(User::class)->create()->id,
            'title' => $faker->sentence,
            'content' => $faker->text
        ];
    });

    //DatabaseSeeder
    $this->call(AdminTableSeeder::class);
```

執行
php artisan db:seed --class=AdminsTableSeeder
或
php artisan db:seed

## 初始化會員系統

laravel 內建一個會員系統，可能是因為太普遍，所以他直接幫你內建一個

    php artisan make:auth

下完指令之後，內建的會員相關檔案都會建好，484 很方便呢
![會員登入畫面](/images/會員登入畫面.png)

laravel 在 6.0 之後把這功能抽出來

    composer require laravel/ui
    php artisan ui vue --auth

執行之後的效果是一樣的

## 自製指令

    php artisan help make:command
    php artisan make:command AddCompanyCommand

接著在`app\Console\Commands\AddCompanyCommand.php`修改

```php
    // 加上?代表該參數可選
    // protected $signature = 'contact:{name} {phone?}';
    // protected $signature = 'contact:{name} {phone=N/A}';
    protected $signature = 'contact:conpany';
    protected $description = 'add new company';
    public function handle()
    {
        $name = $this->ask('What is the company name?');
        $phone = $this->ask('What is the company\'s phone number?');
        if ($this->confirm('Are you ready to insert "' . $name . '"?')) {
            $company = Company::create([
                'name' => $name,
                // 'phone' =>$this->argument('name') ?? 'N/A';
                'phone' => $phone,
            ]);
            return $this->info('Added: ' . $company->name);
        }
        $this->warn('No new company was added.');
    }
```

下 artisan 指令之後就會發現新的 command 出現在列表了
php artisan

contact
contact:company add new company

## 簡單指令

`routes\console.php`用於簡單的指令，盡量不要超過 10 行，最多就像下面的範例

```php
Artisan::command('contact:company-clean', function (){
    $this->info('Cleaning!');
    Company::whereDoesntHave('customers')
        ->get()
        ->each(function ($company) {
            $company->delete();
            $this->warn('Deleted: ' . $company->name);
        });
})->describe('Cleans Up Unused Companies');
```

## 符號連結 (Symbolic link)指令

會將/storage/app/public 和/public/storage 做連結，
切忌路徑就這這兩個，如果要連接其他的路徑，請去網路上找連結的指令
php artisan storage:link
