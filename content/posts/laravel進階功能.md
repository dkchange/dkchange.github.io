---
title: laravel進階功能
date: 2020-10-05 11:02:05
tags:
  - service container
categories:
  - [後端路線, framework, laravel]
---

# symfony

一個開源的框架，因為致力於做出PHP比較底層的原件，所以很多php框架都底層都用他，laravel也不例外，其中尤其重要的是
http-kernel這個元件


# 生命週期
[Laravel 執行原理分析與原始碼分析,底層看這篇足矣](https://learnku.com/articles/54613)

# Service Container

laravel 服務容器管理 class 的依賴關係的一種方式，例如 route model binding，讓你可以依賴注入，
好處是當你的建構子需要改東西時，你不用在個別有 new 他的地方改 code，你只需要改一個地方

https://laracasts.com/series/laravel-6-from-scratch/episodes/39?autoplay=true

那由於 laravel 最自動幫你找命名空間下的 class,有的話他就會幫你注入，你不必自己去綁定進 app 這個 container，但是如果你是個 interface，laravel 就不能幫你實體化了

在 AppServiceProvider

```php
public function register(){
    $this->app->bind(PaymentGateway::class,function($app){
        return new PaymentGateway('usd');
    });
}
```

# View Composer

解決的痛點是，如果各個 controller 都需要用到某個 model 的資料來應用在各個 view 上面，
就不需要在每個 controller 上面都都進行 query，而且當 query 語句要變，你只需要改一個地方

在 AppServiceProvider

```php
public function boot(){
    // option1 每一個view都進行query耗效能
    View::share('channels',Channel::orderBy('name')->get());
    // option2 指定view進行query
    View::composer(['post.*','channel.index'],function($view){
        $view->with('channels',Channel::orderBy('name')->get());
    });
}
```

# Polymorphic Relationships

https://www.itdaan.com/blog/2011/03/22/1861cf1574aa7995b47010f2e11f87e2.html

多態的關聯不是真的表格中的關聯，而是定義在 class 中的關係，程式自己會去找欄位對應的關係
所以不必在建立表格時，設定 fk，也不必而外建立一張表，但要注意這是建立在 eloquent 層，而不是 db 層，如果刪除資料，別的資料並不會有所聯動
https://youtu.be/6J8vb5_WRBw

- 1 對 1 的關係 ，不常見
  例如： 一個人對應一個身份字號
- 1 對多的關係，最常見的一種
  例如： 一個有多筆訂單
- 多對多的關係，在實際的表格設計中你就需要設計另外一個表個來存放關聯
  例如：一個文章有多個標籤，一個標籤屬於多個文章

# Facades

看起來像呼叫靜態方法的設計模式，配合著 service contaier 來做，達到用看似使用靜態方法的 code 來呼叫需要實例化的方法
不用像參數依賴注入一樣不需要在 register 註冊，直接使用更方便，你參數才不會議長串很難看，那跟依賴注入一樣，new 的動作就交給 container

在 AppServiceProvider

```php
public function boot(){
    $this->app->singleton('Postcard',function($app){
        return new PostcardSendingService('us','600','400');
    });

}
```

# Macros

用來擴充 laravel 內建 class 方法 的方式，laravel 官網有列可以這樣擴充的清單

# Pipeline

# Repository Pattern

有人主张 controller 應該要愈少行愈清楚越好，
為了不讓 controller 參雜重複太多邏輯，有些人會把邏輯直接放在 model，讓 orm 的 find where 留在 model，但有時候商業邏輯牽扯到兩個 model 時，你是要將他寫在哪一個 model 呢？
所以有些人會特地抽出一層 repository 來，一切看你專案複雜度而定

好處是假如你哪一天 orm 換掉了，controller 和 model 也不會有太大變動，商業邏輯的部份特地切了出來

中小型專案並不建議導入，會增加專案的複雜度

為什 laravel 不內建這設計模式？
因為業界對於這個設計模式沒有一個準則
https://stackoverflow.com/questions/60295553/creating-laravel-repositories-and-binding-as-service-providers

# Lazy Collection

底層是用 php 的 generator 來達成的，配合 for 和 yield 關鍵字，可以幫你省下不少記憶體，算是蠻少用的功能，但需要時就很實用

```php
Collection::times(100000)->map(function($number){
  return pow(2,$number);
})->all();
User::all();
 // vs
LazyCollection::times(100000)->map(function($number){
  return pow(2,$number);
})->all();
User::cursor();
```

# Soft delete

這也是很常用的功能
大部分的時候，如果你要刪除使用者，你並不會真的刪除他，而是把他隱藏起來
需要在 migration 做欄位的增加，通常會配合 policy 來實作

```php
use Illuminate\Database\Eloquent\SoftDeletes;
class Post extends Model{
  use SoftDelets
  protected $guarded = [];
}
```

# Notifications

通知使用者的機器人，通常會有第三方的套件給你用

# laravel design pattern

https://stackoverflow.com/questions/60029955/when-to-use-repository-vs-service-vs-trait-in-laravel
https://codesource.io/brief-overview-of-design-pattern-used-in-laravel/

