---
title: 提升php
date: 2020-04-29 12:02:24
tags:
categories:
---

# 前言

語言百百種，一開始入門的一個語言是 PHP，中途又跑去碰了 JS，JS 演變非常的快，來來回回耗費了許多生命，但是我只有一條命，是該專注於其中一種語言了，
即便這門語言不是那麼的賺錢

- PHP 葵花寶典
  PHP 葵花寶典：[aWaiMai](https://www.awaimai.com/php)
  PHP 葵花寶典：[博客園](https://www.cnblogs.com/aiweixiao/p/8202365.html)
  PHP 葵花寶典：[Ti-Node](http://t.ti-node.com/thread/6445811931549794305)（連結已失效）
  PHP 葵花寶典：[微信文章](https://mp.weixin.qq.com/s/NTbS3KtToodhFfdjrBjBsw)
  PHP SAPI 解析：[數據挖掘](https://phper.shujuwajue.com/phpxuan-xiang-he-yun-xing-yuan-li/php-sapi)

PHP7 學習筆記：[GitBook](https://imyoungyang.gitbooks.io/php7-study-group-notes/content/packagist.html)

PHP Internals 文檔：[LearnKu](https://learnku.com/docs/php-internals/php7)

- PHP 進階探討
  PHP 進階探討：[藍橋](https://www.lanqiao.cn/library/advanced-php/)

# 什麼是 CGI、FastCGI、PHP-FPM

CGI 與 [[FPM]]
CGI 是一種協議，為了保證 web server 傳過來的資料是標準格式

比如說，如果請求 index.html，web server 會去找到這個文件再丟給瀏覽器，但這個只限於靜態文件而已，如果是 index.php 呢，就需要去找 PHP 解析器來處理了，接下來 web service 就會把這個請求交給 PHP 解析器處理，那會傳哪些資料呢？像是 POST 或是 URL 還有 HTTP header 等，CGI 就是規定要傳哪些資料、以及怎麼樣的格式

FastCGI 是什麼？
接下來又提到那 FastCGI 是什麼呢？FastCGI 是用來提高 CGI 處理 process 性能用的

那 FastCGI 會怎麼實作呢？
當 PHP 啟動時，會去尋找 php.ini、進行環境的初始化，如果不使用 FastCGI 的情況下，每一個請求都會做這個動作，很明顯浪費系統資源，所以 FastCGI 會先啟動一個 master，解析配置文件用，接下來再啟動 worker，當請求過來時，master 會遞資訊給 worker，然後接下來等下一個請求，有這個機制就不用每一次重新跑一次初始化的動作了

PHP-FPM
那 PHP-FPM 呢？FastCGI 是一個協議，其實是 PHP-fpm 實現了這個協議

PHP-fpm 是管理 fastcgi，大概的關係就是這

所以最後才會造成如果修改 php.ini 檔案之後，才需要重新啟動 PHP-fpm，原因就是這個樣子

Nginx FastCGI 詳解：[UniSharp Blog](https://github.com/UniSharp/blog/blob/master/2017/10/04/Nginx%20%E5%AD%B8%E7%BF%92%E7%AD%86%E8%A8%98%EF%BC%88%E4%BA%8C%EF%BC%89FastCGI%20Proxying/index.html)
FastCGI vs PHP-FPM：[Astralweb](https://www.astralweb.com.tw/what-is-differences-between-fastcgi-php-fpm/)
FastCGI 詳解：[zqb520](https://www.zybuluo.com/phper/note/50231)

# PHP 請求外部資源

可以：

- file_get_contents
- fopen
- fsockopen
- curl

# PHP 開啟服務

stream_socket_server()
Workerman 的原理

# 重構

重構簡報：[Google Slides](https://docs.google.com/presentation/d/10vxyi5VXSSZ4MMjLrKAIhrnpmuCjdDaZ_4K7ma1KIys/edit)

## 避免用 switch

PHP Best Practice：[SlideShare](https://www.slideshare.net/kylinfish/php-best-practice-81744253)
PHP Code Style：[LearnKu](https://learnku.com/laravel/t/7468/please-keep-your-php-code-neat-and-tidy)

# Trait

官網講的非常清楚
PHP Trait 官方文檔：[php.net](https://www.php.net/manual/zh/language.oop5.traits.php)
唯一缺的範例是，當 trait 的 function 和要使用的 class 衝突時，你不想覆蓋，想要拿來用

```php
	use FooTrait {
		bar as traitBar;
	}
```

PHP Trait 覆寫：[Andy Carter](https://andy-carter.com/blog/overriding-extending-a-php-trait-method)

# Reflection

獲取檔案路徑 class function params 等等的詳細資料
還可以用來實例一個物件，而非傳統的 new 關鍵字做出實例

# Fun Fact

http_build_query()
http_build_query 問題：[Stack Overflow](https://stackoverflow.com/questions/14761418/browser-mis-interpreting-not-in-url)

# array() vs []

版本問題，5.4 之後推薦使用 [] 為標準用法

# new static()

[PHP new self 、new static 比較](https://xyz.cinc.biz/2016/11/php-new-self-vs-new-static.html)
new self() => 生成的物件為實際寫有這句 code 的 class
new static() =>生成的物件為呼叫使用這句 code 的 class

# PHP_EOL

PHP_EOL 是一個已經定義好的常，代表 PHP 的換行符，這個變數會根據平台而變，在 Windows 下會是 /r/n，在 Linux 下是 /n，在 Mac 下是 /r。
$str = str_replace(PHP_EOL, '', $str);
這段程式碼，在單一來源的輸入沒問題，產出會隨著作業系統做變動，但是
如果
輸入有可能來自不同作業系統，我們沒辦法確定來源作業系統與 server 的相同
那會用到的情境是，我們有使用 RSA 密鑰，但他是 .PEM 金鑰複製貼上的，為了避免 user 的 editor 太過聰明，直接依照作業系統換行，我們後端需要有處理換行的動作
PHP_EOL 處理：[博客園](https://www.cnblogs.com/cxx8181602/p/9132946.html)

$str = str_replace(array("/r/n", "/r", "/n"), "", $str);
換行符處理：[9iPHP](https://9iphp.com/web/php/1006.html)

或是正則
$skuList = preg_split('/\r\n|\r|\n/', $\_POST['skuList']);

# PHP 5.2

RSA256
OpenSSL SHA256：[Stack Overflow](https://stackoverflow.com/questions/10524198/what-version-of-openssl-is-needed-to-sign-with-sha256withrsaencryption)

# PHP 雙引號

PHP 雙引號可以直接帶出變數

```php
echo "hello $name";
//等同於
echo "hello {$name}";
```

# PHP Echo

```php
<?php echo $name; ?>
//等同於
<?= $name ?>
```

# PHP 防止跨站腳本

```php
htmlspecialchars($_GET['userInput']);
```

# PHP 節省 HTML 中的大括號{}

```php
foreach():
endforeach;
if():
endif;
```

# PHP Compact

取變數名組成關聯陣列

# PHP Extract

取關聯陣列 key value 產出變數

# PHP new self, new static

PHP new self vs static：[cinc.biz](https://xyz.cinc.biz/2016/11/php-new-self-vs-new-static.html)
new self() => 生成的物件為實際寫有這句 code 的 class
new static() => 生成的物件為呼叫使用這句 code 的 class
一般就推薦用 static() 才不會有機會實例出父類別

# 為什麼要用 require 而不是 include；為什麼要用 require 而不是 require_once

include 僅會拋出 Warning
require_once ，因為在執行時期完全沒有任何警告或錯誤，以致於造成冗餘程式碼
require 會在引入時發生錯誤時（例如被引入目標不存在）拋出 Error 且終止應用程式
composer autoload 有這 4 種方式
psr0,psr4,classmap,files 自動載入

# 製作 composer package

發布 Composer 套件：[Rivsen](https://rivsen.github.io/post/how-to-publish-package-to-packagist-using-github-and-composer-step-by-step)

# composer replace 取代一個套件

## fork 出沒有人維護的包,用 fork 的包來取代其他有用這個包的依賴

Composer Replace：[Jaceju](https://jaceju.net/composer-replace/)

## 安裝沒有在 packagist 上註冊的私人套件

安裝私人 Composer 套件：[Johnsonlu](https://blog.johnsonlu.org/getting-started-with-composer/)
安裝私人 Composer 套件：[HelloSanta](https://www.hellosanta.com.tw/blog/how-to-add-3rd-party-plugin-by-using-composer)

## 設定私有庫來源，並給予帳密

Composer 私有庫認證：[Stack Overflow](https://stackoverflow.com/questions/35082739/composer-asks-only-for-password-with-private-repository)

# laravel 的方法

Laravel Composer 認證：[Chipperci](https://docs.chipperci.com/builds/composer-auth/)

# composer autoload 的幾種方式

Composer Autoload 方式：[malagege](https://malagege.github.io/blog/2019/01/19/composer%E4%BD%BF%E7%94%A8psr0-psr4-classmap-files%E8%87%AA%E5%8B%95%E8%BC%89%E5%85%A5%E7%B4%80%E9%8C%84/)

# RSA 公鑰

openssl_get_publickey server 回傳 false，但在本機正常，原本以為是作業系統不同的問題，
一查可能是因為 php 版本的問題，為了應付
網路上說

    我知道了 其实公钥是不能放在一行写的，要用原来demo里的，支付宝还告诉我一定要放在一行写，坑爹啊
    一行书写，在windows下是正常的，在linux下返回false

上面寫的不全正確，應為我環境都是 linux 照樣一個報錯一個沒報
server php 5.6.24
local PHP 5.6.40
在這裡筆記一下
每 64 字元換行

# json_encode 出線換行符

```php
json_encode($data,true);
```

即可回傳正常 json

# 列出 PHP 加密演算法列表

print_r(hash_algos());

# PHP Info

commandline php -i

# Xdebug 配置

[root] # cd /usr/local/src
[root] # tar zxvf xdebug-2.6.0RC2.tgz
[root] # cd xdebug-2.6.0RC2
[root] # /usr/local/php/bin/phpize
[root] # ./configure --enable-xdebug --with-php-config=/usr/local/php/bin/php-config
[root] # make && make install
zend_extension=xdebug.so //指定 Xdebug 擴展文件的路徑
xdebug.remote_enable=1 //是否開啟遠程調試
xdebug.remote_handler=dbgp //指定遠程調試的處理協議
xdebug.remote_mode=req //可以設為 req 或 jit，req 表示腳本一開始運行就連接遠程客戶端，jit 表示腳本出錯時才連接遠程客戶端。
xdebug.remote_host=192.168.1.98 //指定遠程調試的主機名（安裝 phpstorm 的主機 ip）
xdebug.remote_port=9001 //指定遠程調試的端口號
xdebug.idekey="PHPSTORM" //指定傳遞給 DBGp 調試器處理程序的 IDE Key
Xdebug 配置並不簡單
Xdebug 配置教學：[SegmentFault](https://segmentfault.com/a/1190000011907425)
Xdebug 配置教學：[SegmentFault](https://segmentfault.com/a/1190000018961750)
Xdebug 配置教學：[一速科技](https://www.yisu.com/zixun/39716.html)

# disable_function

disable_function 安全性：[安全客](https://www.anquanke.com/post/id/197745)

# PHP Lint

雖然 PHPStorm 內建整合 PHP-CS-Fixer ，跟 Laravel 一樣的套件
PHP-CS-Fixer：[Laracasts](https://laracasts.com/discuss/channels/laravel/php-cs-config-for-php-cs-fixer)
阿我們還是希望協作者能夠在提交前跑一次格式檢查
此時就會用到
使用 Git pre-commit 自動修正 PHP 的 Coding Style
Git Pre-commit PHP 格式：[CodeFun](https://codefun.tw/2019/2019051901-php-coding-style-fix-with-git)

# 字串比較

mbstring 使用了國家默認語言設置（NLS），所以在做字串長度比較時，需要注意
mbstring.language = UTF-8
mbstring.internal_encoding = UTF-8

# fastcgi_finish_request

當 PHP 運行在 FastCGI 模式時，PHP FPM 提供一個名為 fastcgi_finish_request 的方法。按照文檔上的說法，此方法可以提高請求的處理速度，如果有些處理可以在頁面生成完後再進行，就可以使用這個方法

fastcgi_finish_request：[火丁](https://blog.huoding.com/2011/04/12/63)

# 精度問題

所有程式語言一定都有的問題，因為二進位無法完美表示浮點數
PHP 官方建議如果需要高位數的計算的話，需要使用 BC Math 或是 GMP
[關於 PHP 浮點數 float 以及 int 的問題](https://blog.walile.info/2013/06/14/about-php-float-problem/)

# PHP sleep() 是否會佔用很多資源？

sleep 本身不佔 CPU 資源，但是在基於 PHP 的 LAMP 環境中，就不能這樣說了，因為 CPU 不是唯一的資源。進程數、記憶體，這些都是資源。

# PHP 運用多執行緒

PHP 可以利用其他的東西來實現偽多進程，多線程，例如：fsockopen 實際是利用 socket 的多線程，popen，pcntl_fork，proc_open 利用 httpd 多進程功能的外衣
[《面試官別再問》PHP 運用多執行緒(Multi-thread)實現非阻塞方法](https://bps1025.blogspot.com/2018/10/php-multi-threadprocopen.html)
[How do you make good use of multicore CPUs in your PHP/MySQL applications?](https://stackoverflow.com/questions/2267345/how-do-you-make-good-use-of-multicore-cpus-in-your-php-mysql-applications)

# PHP+MySQL 多語句執行

PHP MySQL 多語句：[精靈鼠](http://www.jinglingshu.org/?p=3941)
其實 MySQL 早在 4.1 版本就允許多語句執行。只是 PHP 自身限制了這種用法。

# Composer Autoload 會佔據記憶體嗎

答：不會

如果下面連結說的是正確的話，會在實際 new 的時候才載入 CODE，記憶體一開始 autoload 只有一個檔案位址的陣列
Composer Autoload 記憶體：[Stack Overflow](https://stackoverflow.com/questions/37283217/composer-dont-use-autoload-and-load-single-classes)

# 效能調校

- where: local 端測試
- when: testing, deploying
- what: 執行時間，記憶體使用量
- 目標: 在效能與可維護之間做平衡
  PHP 效能調校：[YouTube](https://www.youtube.com/watch?v=hOajLLej68Y)

# & before the function name

```php

 class FooBar {
     private $properties = array();

     public function &__get($name) {
         return $this->properties[$name];
     }

     //If I hadn't used & there, this wouldn't be possible:
     public function __set($name, $value) {
         $this->properties[$name] = $value;
     }
 }

 $foobar = new FooBar;
 $foobar->subArray = array();
 $foobar->subArray['FooBar'] = 'Hallo World!';
```

Instead PHP would thrown an error saying something like 'cannot indirectly modify overloaded property'.
PHP & 函式名：[Stack Overflow](https://stackoverflow.com/questions/3255516/what-does-before-the-function-name-signify)
PHP & 物件：[Stack Overflow](https://stackoverflow.com/questions/21058439/is-there-ever-a-need-to-use-ampersand-in-front-of-an-object)

# PHP 強制使用「強型態」的模式

```php
declare(strict_types = 1);
```

# json_encode() 浮點小數溢位錯誤

該現象只出現在 PHP 7.1+ 版本上
這個就是只有靠經驗才會知道的知識點
可以靠 serialize_precision 這個解決

# PDO 和 MySQLi

這兩個庫都是很熱門的資料庫 API，各有優缺點，但 PDO 不只支援 MySQL

[PDO vs. MySQLi: The Battle of PHP Database APIs](https://websitebeaver.com/php-pdo-vs-mysqli)

# 假設你要測量一個 PHP function 的效能，要如何在不影響原有代碼的情況下，標記執行時間呢？

PHP 函式效能標記：[Stack Overflow](https://stackoverflow.com/questions/9262158/how-can-i-inject-and-remove-php-code-before-a-function-call)

# json_deocde

在接收兩個幾乎相同的字串時，php json_decode 會報出 syntax error的錯
錯的原因，雙引號前面多了個slash，奇怪的是我單純放進input後端magic quote產生有slash的字串不會錯，但是前端JSON.Stringfy()之後放進input 後端產生有slash的字串卻抱錯了
JSON Decode Error：[Stack Overflow](https://stackoverflow.com/questions/14757983/json-post-with-magic-quotes-plus-a-quote-character-in-the-data)
