---
title: laravel使用事件和監聽
date: 2019-12-17 16:03:31
tags: Events & Listeners
categories:
- [後端路線,framework,laravel]
---
# 前言
使用場景這樣的，有的時候controller使用者註冊完之後，除了要發信件給管理者，還有發LINE訊息給管理者，這些眾多需要做的事都塞在controller會讓controller很雜亂，整個不隸屬與該controller的兩件事，寫同一個controller裡面也不對，萬一很多地方要用到呢?
# 創建事件建和監聽

 php artisan make:event NewCustomerHasRegisteredEvent

`app\Events\NewCustomerHasRegisteredEvent.php`

 ```php
    public $coustmer;

    public function __construct($coustmer)
    {
        $this->customer = $coustmer;
    }
 ```
 php artisan make:listener WelcomeNewCustomer

 `app\Listeners\WelcomeNewCustomer.php`

 ```php
    public function handle($event)
    {
        sleep(10);
        Mail::to($event->customer->email)->send(new WelcomeNewUserMail());
    }
 ```
 # 註冊此事件監聽
 `/app/Providers/EventServiceProvider.php`
```php
     protected $listen = [
        NewCustomerHasRegisteredEvent::class => [
            \App\Listeners\WelcomeNewCustomerListener::class,
        ],
    ];
```
在這裡有一個方便的點就是，如果$listen陣列內容填的是全路徑的話
直接

    php artisan event:ganerate
就可以產生下面兩個檔案了，完全不用分別創建事件和監聽，如果事件和監聽很多的話，少打不少指令

`app\Events\NewCustomerHasRegisteredEvent.php`
`app\Listeners\WelcomeNewCustomer.php`

 # 觸發事件
 在要使用的controller裡面添加

 ```php
 event(new NewCustomerHasRegisteredEvent($customer));
 ```

 # Queues
 有的時候事件的服務要經過很久才會有回應，總不能讓使用者等10秒以上吧，laravel有解決方法。話說PHP又沒有異步機制，要異步的話必須要靠別的線程?laravel如何實現呢?看樣子是利用排程去做
 異步要依靠列隊

 `.env`
 QUEUE_CONNECTION=database
`app\Listeners\WelcomeNewCustomer.php`
 ```php
class WelcomeNewCustomerListener implements ShouldQueue
 ```

    php artisan queue:table
    php artisan migrate
    php artisan queue:work
    要在背景執行的話在後面加上sapce+&

    php artisan queue:work &
    列出執行中的排程

    jobs -l 
    要儲存log的話

    php artisan queue:work > storage/logs/jobs.log &

 排程執行失敗要怎麼知道?
 這時候就要加個工具叫做supervisor排程關掉時他會替你重開
