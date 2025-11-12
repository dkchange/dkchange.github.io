---
title: laravel資訊安全
date: 2021-12-30 10:24:05
tags:
categories:
- [漏洞]
---

# 前言
[How your Laravel application can get hacked, and how to prevent that from happening by Antti Rössi](https://www.youtube.com/watch?v=kKGGVGiq2y8)

# sql injection
如果你用DB::raw裡面直接用unsanitize過的參數，就可以看到整個DB
測試工具 sqlmap 幫你測試一段URL的所帶的參數有沒有機會SQL injection

# object injection
測試工具 phpggc 一個生成序列化的工具，例如可以幫你把一段序列化之後的CODE塞進JPG檔，並生成出來
反序列化一個物件時，他是從字串轉成物件，當你從字串轉成物件時，就有機會被注入
舉個極端的例子  
gadget : 利用
例子1:
```php

https://kylingit.com/blog/%E7%94%B1phpggc%E7%90%86%E8%A7%A3php%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/
https://ithelp.ithome.com.tw/articles/10241555
public $arg = 'id';
public function __destruct(){//php會自動執行
    shell_exec($this->arg);
}
```
例子2:
phar 文件是一種打包格式，通過將許多PHP代碼文件和其他資源（例如圖像，樣式表等）捆綁到一個歸檔文件中來實現應用程式和庫的分發,受到Java的JAR文件格式的影響，，目的在於加快通過FTP部署應用程式的速度
phar在使用任何檔案相關的操作時(e.g. filesize())，會將資料反序列化

例子3:
當含有序列化code的圖片被傳至server時，server去讀它的filesize就會有被駭的風險

# laravel漏洞

[CVE-2021-39165 从一个Laravel SQL注入漏洞开始的Bug Bounty之旅](https://www.leavesongs.com/PENETRATION/cachet-from-laravel-sqli-to-bug-bounty.html)



