---
title: laravel使用model
date: 2019-12-16 17:11:47
tags:
  - model
categories:
  - [後端路線, framework, laravel]
---

# 前言

這篇重點在紀錄 model 使用時的幾個知識點
youtue 上有教學講得很清楚，這邊是看完教學的心得紀錄
參考自:https://youtu.be/L1E_uJxNC3M

laravel 框架中的 Model 操作数据库 , 相比 DB 类有什么明显的优越性吗?
https://www.zhihu.com/question/49649216

# 路由和 model 綁定

Route Model Binding

`/customers/{customer}`路由的參數只要和 founction 參數一樣的變數名稱

```php
   public function show($customer)
    {
        //when the value is not found,this will return 404
        $customer = Customer::where('id',$customer)->firstOrFail();
        return view('customers.show', compact('customer'));
    }
```

就可以省略成這樣

```php
   public function show(Customer $customer)
    {
        return view('customers.show', compact('customer'));
    }
```

# 知識點

## Eloquent Scopes

這個是用來客製化查詢結果的
例如在 model 裡

```php
    public function scopeActive($query)
    {
        return $query->where('active', 1);
    }
```

controller 裡面就可以用**靜態方法**的方式調用

```php
    ModelName::Active()->get();
```

好處就是 sql 的查詢語句都放在 model 裡，controller 程式語意一目瞭然

## Mass Assignment

用在**欄位驗證**過後，需要多欄位寫入資料庫

```php
    // 需要填入的欄位
    // protected $fillable = ['name', 'email', 'active'];
    // 需要禁止的欄位
    protected $guarded = [];
```

以上兩個擇一即可，通常使用 guarded 指定空陣列
然後 controller 就可以直接 create，不用先 new 然後再指定個別欄位屬性再 save

```php
var

```

```php
$data = request()->validate([]);
ModelName::create($this->validateRequest($data));
```

## 表格關聯性

Eloquent BelongsTo & HasMany
添加外來鍵`/database/migrations/create_customers_table.php`，有既定的命名規則表名+ID

```php
    public function up()
    {
        Schema::create('customers', function (Blueprint $table) {

            $table->unsignedInteger('company_id');

        });
    }
```

`/app/Company.php`

```php
    public function customers()
    {
        return $this->hasMany(Customer::class);
    }
```

`/app/Customer.php`

```php
    public function company()
    {
        return $this->belongsTo(Company::class);
    }
```

## 表格創建 migration

以下列出常用的方法 nullable 等

```php
    public function up()
    {
        Schema::create('customers', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->unsignedInteger('company_id');
            $table->string('name');
            $table->string('email');
            $table->integer('active');
            $table->string('image')->nullable();
            $table->timestamps();
        });
    }
```

## 避免 N+1query

N+1 query 言下之意就是查 N 筆資料，用了 N+1 次查詢。
這個問題呢會出現在關聯式資料庫，尤其我們在用 ORM 時，不小心就會
select All 這個 query 之後再用 for 迴圈跑 N 次 qeruy

那既然是不小心的，有沒有辦法可以察覺?
laravel 有提供一個工具叫作 Telescope，跟著官網作，開發階段一旦發現有相同的網址請求太多次，應該就是犯了這個錯了

解決問題

```php
//因為我們要的欄位關聯自別的表，當for迴圈訪問到該欄位時，因為找不到該屬性，laravel就會去query這個屬性出來
$customers = Customer::all();
//改成
$customers = Customer::with('company')->get();

```

這樣就可以回復成 1 條 query 了

## 分頁

資料庫常常會有需要分頁查詢的情形

```php
$customers = Customer::with('company')->pagenate(15);

//在view這邊，用以下的方法會直接產生對應的html
$customers->links();

```

## 權限存取

我們常會有需要管理員、編輯、訪客等不同權限的需求來對資料作存取，在 laravel 就是對 model 作權限控管

### 創建 policy

    // -m加上要控管的model名字
    php artisan make:policy CustomerPolicy -m Customer

### 使用方法

`app/Policies/CustomerPolicy.php`

```php
    public function view(User $user, Customer $customer)
    {
        return in_array($user->email, [
            'admin@admin.com',
        ]);
    }
```

`app/Http/Controllers/CustomersController.php`

```php
$this->authorize('create', Customer::class);
```

或是

`/routes/web.php`

```php
Route::get('customers/{customer}', 'CustomersController@show')->name('customers.show')->middleware('can:view,customer');
```

`/resources/views/customers/index.blade.php`

```php
    @can('create', App\Customer::class)
        <div class="row">
            <div class="col-12">
                <p><a href="{{ route('customers.create') }}">Add New Customer</a></p>
            </div>
        </div>
    @endcan

    @cannot
    <p> not allow to add </p>
    @endcannot
```

## 關聯介紹 1 對 1

1 to 1 這個使用場景並不常見
一個使用者有一個電話

```php

    public function phone()
    {
        return $this->hasOne(Phone::class);
    }

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    public function up()
    {
        Schema::create('phones', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('phone');
            $table->unsignedBigInteger('user_id');
            $table->timestamps();
            $table->foreign('user_id')->references('id')->on('users');
        });
    }
    //這一個待測試
    // $phone = new \App\Phone();
    // $phone->phone='222-333-4567';
    // $phone->user_id=1;
    // $phone->save();
    //或
    $user = factory(\App\User::class)->create();
    $phone = new \App\Phone();
    $phone->phone='222-333-4567';
    $user->phone()->save();
    //或
    $user = factory(\App\User::class)->create();
    $user->phone()->create([
        'phone' => '222-333-4567',
    ]);
```

## 關聯介紹 1 對多

1 to many 最常見

```php
    public function user()
    {
        return $this->belongsTo(\App\User::class);
    }
    public function posts()
    {
        return $this->hasMany(\App\Post::class);
    }
    public function up()
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->unsignedBigInteger('user_id');
            $table->string('title');
            $table->text('body');
            $table->timestamps();
        });
    }

    $user = factory(\App\User::class)->create();
    $post = new \App\Post(
        'title' => 'Title Here',
        'body' => 'body Here',
        'user_id'=> $user->id
    );
    $post->save();
    //或
    $user = factory(\App\User::class)->create();
    $user->posts()->create([
        'title' => 'Title Here',
        'body' => 'body Here',
    ]);

    //更新
    $post = $user->post-first();
    $post->title = 'New Title';
    $post->body = 'New Better Body';
    $post->save();
    //或
    $user->posts->first()->title = 'New Title';
    $user->posts->first()->body = 'New Better Body';
    $user->push();
```

## 關聯介紹多對多

什麼情況下會用到 N to N 呢?
有一種情況是權限列表，我使用者有多個權限，權限又有多個使用者，此時再建一個表來對應權限和使用者，就會是多對多的情況，那關聯式資料庫的好處就是你刪掉一個，他會提醒你另外一張表也要刪，或是一起刪掉

那通常情形是這樣的，我會除了這張對應的關係表，還會再作一張角色的表，這張表只對應到權限，用來給客戶快速選取用，實際作權限判斷用不到這方面相對複雜，詳情請看前言的 youtube 連結，有老師把把手把手帶

# api 輸出格式化

有的時候們資料庫撈出來的東西還需要再格式化，例如日期格式化，這這方面就不寫在 controller 了
php artisan make:resource Contact

# 遞迴表格

https://stackoverflow.com/questions/26652611/laravel-recursive-relationships
