---
title: laravel使用middleware
date: 2019-12-17 15:13:08
tags:
  - middleware
categories:
  - [後端路線, framework, laravel]
---

# 前言

Middleware 是 Laravel 請求生命周期中的「过濾層」，每個進入 controller 的 HTTP 請求都會先經過 middleware 處理。常見用途如：身份驗證、記錄日誌、CSRF 防護等。

本篇以建立管理後台登入驗證為範例，相關登入配置可參考 [[laravel會員登入配置]]。

# 建立 Middleware

```bash
php artisan make:middleware AdminMiddleware
```

# 在 Kernel.php 註冊

## 全域 Middleware（$middleware）

`app/Http/Kernel.php` 的 `$middleware` 處理的是全域 middleware，每個 request 都會經過。

```php
protected $middleware = [
    \App\Http\Middleware\CheckForMaintenanceMode::class, // Laravel 7 以前，8+ 改為 PreventRequestsDuringMaintenance
    \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
    \App\Http\Middleware\TrimStrings::class,
    \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
    \App\Http\Middleware\TrustProxies::class,
];
```

## 路由群組 Middleware（$middlewareGroups）

`$middlewareGroups` 群組中的 middleware 只會套用於對應路由群組。例如下面的 `web` 群組 middleware 只會套用於 `web.php` 裡面的路由。

```php
'web' => [
    \App\Http\Middleware\EncryptCookies::class,
    \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
    \Illuminate\Session\Middleware\StartSession::class,
    // \Illuminate\Session\Middleware\AuthenticateSession::class,
    \Illuminate\View\Middleware\ShareErrorsFromSession::class,
    \App\Http\Middleware\VerifyCsrfToken::class,
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
],
```

## 路由別名 Middleware（$routeMiddleware）

註冊在 `$routeMiddleware` 裡面，帶別名的路由可以給 route 使用，不註冊的就需要全部路徑。

```php
protected $routeMiddleware = [
    'admin.auth' => \App\Http\Middleware\AdminMiddleware::class,
];
```

# 使用方法

## 在 Controller 中使用

在建構式中透過 `$this->middleware()` 套用，可以用 `->except()` 或 `->only()` 指定排除或限制的方法。

`Controllers\Admin\LoginController.php`

```php
public function __construct()
{
    // 此 controller 內所有方法都需要登入，但登入表單頁面本身除外
    $this->middleware('admin.auth')->except(['loginForm', 'login']);
}
```

## 在 Route 中使用

`web.php`

```php
Route::group(['prefix' => 'admin'], function () {
    Route::get('/', [\App\Http\Controllers\Admin\LoginController::class, 'loginForm']);
    Route::post('/login', [\App\Http\Controllers\Admin\LoginController::class, 'login']);
    Route::get('/home', [\App\Http\Controllers\Admin\LoginController::class, 'home']);
    Route::post('/logout', [\App\Http\Controllers\Admin\LoginController::class, 'logout']);
    Route::get('/resetPassword', [\App\Http\Controllers\Admin\ProfileController::class, 'showResetPasswordForm']);
    Route::post('/resetPassword', [\App\Http\Controllers\Admin\ProfileController::class, 'resetPassword']);
})->middleware('admin.auth');
```

> **注意**：Laravel 8+ 已廢棄 `Route::group` 中的 `namespace` 參數，請改用完整類別命名空間（如上范例）或在 `RouteServiceProvider` 中統一配置。