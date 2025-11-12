---
title: laravel使用middleware
date: 2019-12-17 15:13:08
tags: 
- middleware
categories:
- [後端路線,framework,laravel]
---
# 前言
laravel middleware用於驗證登入方面，請見{% post_link laravel會員登入配置 會員登入配置 %}
# 創建middleware
php artisan make:middleware AdminMiddleware
# 使用方法
## 文件介紹
`app\Http\Kernel.php`屬性介紹
每個請求都要經過註冊在***$middleware***裡面的middleware
```php
    protected $middleware = [
        \App\Http\Middleware\CheckForMaintenanceMode::class,
        \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
        \App\Http\Middleware\TrimStrings::class,
        \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
        \App\Http\Middleware\TrustProxies::class,
    ];
```
註冊在***$middlewareGroups***裡面的middleware會被應用至該路由，例如下面幾個middleware都會被應用至web.php裡面的路由
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

註冊在$routeMiddleware並給名字的話可以給route用，不註冊的話就要寫全部路徑
```php
    protected $routeMiddleware = [
        'admin.auth'=>\App\Http\Middleware\AdminMiddleware::class,
    ];
```



## 使用方法

以下為中間件的食用方式，添加在建構子，或是個別路由
`Controllers\Admin\LoginController.php`
```php
    public function __construct()
    {
        //必須要略過登陸頁路由，因為我們需要給未登入的人用爾且我們驗證失敗會倒到這裡，會形成無窮迴圈
        //必須要略過登陸驗證路由，因為我們需要給未登入的人用
        $this->middleware('admin.auth')->except(['loginForm','login']);
    }  
```
`web.php`
```php
    Route::group(['namespace' => 'Admin'], function () {
        Route::get('/','LoginController@loginForm');
        Route::Post('/login','LoginController@login');
        Route::get('/home','LoginController@home');
        Route::post('/logout','LoginController@logout');
        Route::get('/resetPassword','ProfileController@resetPasswordForm');
        Route::post('/resetPassword','ProfileController@resetPassword');
    })->middleware('admin.auth');
```