---
title: laravel會員登入配置
date: 2019-11-07 20:46:04
tags: 後台管理
categories:
  - 後端路線
  - framework
  - laravel
---

# 前言

鑑於我們已經有會員機制，因此 Laravel 就幫我們準備好了

> **適用版本**：Laravel 5.x ~ 7.x。Laravel 8+ 建議使用 [Laravel Breeze](https://laravel.com/docs/8.x/starter-kits#laravel-breeze) 或 [Jetstream](https://jetstream.laravel.com/)。

# make:auth

下完這個指令，[[Laravel]] 自動幫會員登入系統

```bash
php artisan make:auth
```

Laravel 在 6.0 之後就需要另外外套件了

```bash
composer require laravel/ui
php artisan ui vue --auth
```

實作之後的效果是一樣的
路由就會多了 `Auth::routes()`；這個方法事實上就是我們要加以下面的路徑

```php
// Authentication Routes...
$this->get('login', 'Auth\LoginController@showLoginForm')->name('login');
$this->post('login', 'Auth\LoginController@login');
$this->post('logout', 'Auth\LoginController@logout')->name('logout');

// Registration Routes...
$this->get('register', 'Auth\RegisterController@showRegistrationForm')->name('register');
$this->post('register', 'Auth\RegisterController@register');

// Password Reset Routes...
$this->get('password/reset', 'Auth\ForgotPasswordController@showLinkRequestForm');
$this->post('password/email', 'Auth\ForgotPasswordController@sendResetLinkEmail');
$this->get('password/reset/{token}', 'Auth\ResetPasswordController@showResetForm');
$this->post('password/reset', 'Auth\ResetPasswordController@reset');
```

要對這些路由對應 Controller 做修改或 Override 即可，例如：
放到登入畫面
`Controllers/Auth/LoginController.php`

```php
public function showLoginForm()
{
    return view('admin.login');
}
```

怎樣我們都需要儲存，用不同的表放置就登入人嗎？
這問題可以參考這裡的討論
[Stack Overflow 討論](https://stackoverflow.com/questions/4169893/is-it-good-database-design-to-have-admin-users-in-the-same-table-as-front-end-us)
[超全面的權限系統設計方案！](https://mp.weixin.qq.com/s/wox0f-bl9F7gAl3pXuuMTg)

本篇主要介紹後端操作後面頁面

# 建立資料表

```bash
php artisan make:model Admin -a
```

設計好欄位

```bash
php artisan migrate
```

# 建資料

後台不會有註冊功能，所以比較理想的方法是用 Admin 還是管理者 Seeder

```php
Admin::create([
    'username' => 'admin',
    'password' => Hash::make('password'),
    'api_token' => Str::random(32), // Laravel 5.x Token Guard，8+ 請改用 Sanctum
]);
```

```bash
php artisan db:seed
```

# 使用名和密碼就能登入

因為 Laravel 有內建登入功能了，
所以我們只要後台的就算順利登入即可

## config/auth.php

添加下面的 `code` 在 `guards`、`providers` 這兩個 key 下面

```php
'guards' => [
    'admin' => [
        'driver' => 'session',
        'provider' => 'admins', // 對應下面provider的admins
    ],
],
'providers' => [
    'admins' => [
        'driver' => 'eloquent',
        'model' => App\Admin::class,
    ],
],
```

## Controllers/Admin/LoginController.php

建立 Controller

```bash
php artisan make:controller Admin/LoginController
```

以下為使用方式

```php
// Controller
use Auth;

// 驗證方式對登 admin，配合 config/auth 設定的 provider 進行驗證
Auth::guard('admin')->attempt([
    'username' => $request->input('username'),
    'password' => $request->input('password')
]);
```

## App/Admin.php

因為 Admin Model 還沒有驗證功能，所以需要繼承 `Illuminate\Foundation\Auth\User`（即 Laravel 內建的 Authenticatable 基礎類別）

```php
use Illuminate\Foundation\Auth\User as Authenticatable;

class Admin extends Authenticatable
{
}
```

設定完就順利登入了

# 防止未登入使用，轉跳登入頁面

為防止使用者直接存取後台頁面，我們對每一個請求都要進行驗證，這就是 Middleware 需要中間件了

詳細可參考 [[laravel使用middleware]]

## Middleware/AdminMiddleware.php

建立 Middleware

```bash
php artisan make:middleware AdminMiddleware
```

```php
if (!Auth::guard('admin')->check()) {
    return redirect('/');
}
```

## app/Http/Kernel.php

```php
protected $routeMiddleware = [
    'admin.auth' => \App\Http\Middleware\AdminMiddleware::class,
];
```

## Controllers/Admin/LoginController.php

以下為 Middleware 的使用方式，添加在建構式中，或是個別路由

```php
public function __construct()
{
    // 此 controller 內所有方法都需要登入，但登入表單頁面本身除外
    $this->middleware('admin.auth')->except(['loginForm', 'login']);
}
```

這還有一個小注意，每次我們用 Auth 的時候都需要調用 guard('admin')，將這些 [GitHub 問題](https://github.com/laravel/framework/issues/19547)

```php
// 放在就能命令行可，這樣篩選
// Auth::shouldUse('admin');
```
