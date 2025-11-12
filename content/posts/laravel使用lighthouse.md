---
title: laravel使用lighthouse
date: 2019-11-15 20:20:12
tags: 
- graphql
- postman
categories:
- [後端路線,framework,laravel]
---
# 介紹
因為graphql太紅，所以要來找找laravel上面如何應用，[lighthouse](https://lighthouse-php.com/)就是其中一款比較熱門的套件，那在應用上就要多看官網的文件了，遇到坑會再更新上來

# 安裝步驟
跟著官網做就會成功，除了1G的記憶體composer跑不起來以外，沒什大問題

# 客製化查詢
如果遇到需要太複雜，內建指令無法應付，客製化查詢的語句，就可以建一個class來搭配eloquent做應用

    php artisan lighthouse:query customerQuery
# 測試工具
神器postman支援graphql囉，如果本機是6.X的版本要煮度上網去下載7.X的，自動更新不會幫你更到7.X

# laravel 跨站設定
最簡單的方法就是

    composer require barryvdh/laravel-cors

然後進到

    config\lighthouse.php
    
```php
    'middleware' => [
        \Nuwave\Lighthouse\Support\Http\Middleware\AcceptJson::class,
        \Barryvdh\Cors\HandleCors::class
    ]
```
# 文件上傳設定
跟著lighthouse官網做，其中測試檔案上傳 -k允許不安全連線

    curl -k https://dkchange.nctu.me/graphql   -F operations='{ "query": "mutation ($file: Upload!) { upload(file: $file) }", "variables": { "file": null } }'   -F map='{ "0": ["variables.file"] }'   -F 0=@my_file.txt
resolver那邊laravel
$file->storePublicly('uploads')會出現在`myproject/storage/app/uploads/`
那使用者怎麼存取?
artisan 提供一個指令

php artisan storage:link
下完之後就會把sotrage的public資料夾複製一分到public底下，阿記得這種方式沒有權限控管





