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
安裝時請確保系統記憶體大於 1GB，否則 composer install 可能因記憶體不足而失敗。

# 客製化查詢
如果遇到需要太複雜，內建指令無法應付，客製化查詢的語句，就可以建一個class來搭配eloquent做應用

    php artisan lighthouse:query customerQuery
# 測試工具
測試時需使用 Postman，若使用舊版 6.x 需手動升級到 7.x 才能支援 GraphQL，因若自動更新不會主動提示升級。

# laravel 跨站設定
最簡單的方法就是

    composer require barryvdh/laravel-cors
    # 注意：此套件適用於 Laravel 5.x~6.x；Laravel 7.x 之後請改用 fruitcake/laravel-cors；Laravel 9.x 以上 CORS 已內建，不需安裝額外套件。

然後進到

    config\lighthouse.php
    
```php
    'middleware' => [
        \Nuwave\Lighthouse\Support\Http\Middleware\AcceptJson::class,
        \Barryvdh\Cors\HandleCors::class
    ]
```
# 文件上傳設定
測試 lighthouse 檔案上傳功能，以下 curl 指令使用 -k 略過 SSL 憑證驗證（僅限本機測試，生產環境不可使用 -k）：

    curl -k https://dkchange.nctu.me/graphql   -F operations='{ "query": "mutation ($file: Upload!) { upload(file: $file) }", "variables": { "file": null } }'   -F map='{ "0": ["variables.file"] }'   -F 0=@my_file.txt
resolver那邊laravel
$file->storePublicly('uploads')會出現在`myproject/storage/app/uploads/`
那使用者怎麼存取?
artisan 提供一個指令

php artisan storage:link
執行後會在 public 目錄下建立一個指向 storage/app/public 的**符號連結（symbolic link）**，注意這不是複製檔案，因此仍需確認目錄權限設定正確（如 storage 資料夾需有寫入權限）。





