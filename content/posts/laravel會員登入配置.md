---
title: laravel會員登入配置
date: 2019-11-07 20:46:04
tags: 後台管理
categories:
- [後端路線,framework,laravel]
---
# 前言
幾乎每個網站都會有會員機制，所以laravel就幫我們串好了
# make:auth
下完這段指令，laravel自動建會員登入系統

    php artisan make:auth

laravel在6.0之後把這功能抽出來

    composer require laravel/ui
    php artisan ui vue --auth
執行之後的效果是一樣的
路由就會多了Auth::route();這個方法事實上就是幫我們添加以下的路由

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

要客製前台的話，就近去個別controller做修改或Override即可，像是
改會員登入畫面
Controllers\Auth\LoginController.php


```php
    public function showLoginForm()
    {
        return view('admin.login');
    }
```
那如果我們要建個後臺，用不同的表儲存管理人員呢?
這問題牽扯到使用者和管理這484相同的生物了，例如銀行，我們可以看看大家的討論
[Is it good database design to have admin users in the same table as front-end users?](https://stackoverflow.com/questions/4169893/is-it-good-database-design-to-have-admin-users-in-the-same-table-as-front-end-us)
[超全面的权限系统设计方案！](https://mp.weixin.qq.com/s/wox0f-bl9E7qAl3pXuuMTg)
本篇主要探討製作後台頁面


# 創建資料表

    php artisan make:model Admin -a
    設定好欄位
    php artisan migrate
# 塞資料
後台不像前台有註冊頁面，所以，比較合理的方法就是把admin這些管理者寫進seed
```php
// DB::table('admins')->insert([
//     'username' => 'admin',
//     'password' => Hash::make('password'),
//     'api_token'=> Str::random(32),
//     'created_at' => Carbon::now()->format('Y-m-d H:i:s'),
//     'updated_at' => Carbon::now()->format('Y-m-d H:i:s')
// ]);
// Auto timestamp saving is only for Eloquent feature so you need to do manually like below for non eloquent feature
Admin::create([
    'username' => 'admin',
    'password' => Hash::make('password'),
    'api_token'=> Str::random(32), 
]);
```

    php artisan db:seed

# 使用者和資料庫做認證
因為laravel 有內建認證會員了，
所以我們只要後台的會員認證即可
    
## config\auth.php
添加下面的code進guards、providers這兩個key下面

```php
    'guards' => [
        'admin' => [
            'driver' => 'session',
            'provider' => 'admins',//對應下面provider的admins
        ],
    ],
    'providers' => [
        'admins' => [
            'driver' => 'eloquent',
            'model' => App\Admin::class,
        ],
    ],
```



## Controllers\Admin\LoginController.php
建controller

    php artisan make:controller Admin/LoginController
以下為食用方式
```php
    //Controller
    use Auth;

    //驗證方式切到admin，配合config/auth設定的prvider進行驗證
    Auth::guard('admin')->attempt([
            'username' => $request->input('username'),
            'password' => $request->input('password')
        ]);
```

## App\Admin.php
因為admin model並沒有驗證功能，所以需要繼承Illuminate\Foundation\Auth\User
```php
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class Admin extends Authenticatable
    {

    }
```

設定完驗證就能用了


# 防止未登入使用者，繞過登入頁直接存取
為防止使用者直接存取後台頁面，我們每一個請求都要做驗證，那這個時候就需要中間件了

## Middleware\AdminMiddleware.php
建middleware
     php artisan make:middleware AdminMiddleware

```php
    if (!Auth::guard('admin')->check()) {
        return redirect('/');
    }
```

## app\Http\Kernel.php
```php
    protected $routeMiddleware = [
        'admin.auth'=>\App\Http\Middleware\AdminMiddleware::class,
    ];
```

## Controllers\Admin\LoginController.php
以下為中間件的食用方式，添加在建構子，或是個別路由

```php
    public function __construct()
    {
        //必須要略過登陸頁路由，因為我們需要給未登入的人用爾且我們驗證失敗會倒到這裡，會形成無窮迴圈
        //必須要略過登陸驗證路由，因為我們需要給未登入的人用
        $this->middleware('admin.auth')->except(['loginForm','login']);
    }  
```
這邊有個省事的小秘訣，每次要用Auth的時候都需要先調用guard('admin')，有個[直接切換的方法](https://github.com/laravel/framework/issues/19547)

    //放在route裡面判斷即可，超省事
    Auth::shouldUse('admin');

