---
title: 提升php
date: 2020-04-29 12:02:24
tags:
categories:
---

# 前言

語言百百種，一開始入門的一個語言是 php，中途又跑去碰了 js，js 演變非常的快，來來回回耗費了許多生命，但是我只有一條命，是該專注於其中一種語言了，
即便這門語言不是那麼的賺錢

- php 葵花寶典
  https://www.awaimai.com/php
  https://www.cnblogs.com/aiweixiao/p/8202365.html
  http://t.ti-node.com/thread/6445811931549794305
  https://mp.weixin.qq.com/s/NTbS3KtToodhFfdjrBjBsw
  https://phper.shujuwajue.com/phpxuan-xiang-he-yun-xing-yuan-li/php-sapi

https://imyoungyang.gitbooks.io/php7-study-group-notes/content/packagist.html

https://learnku.com/docs/php-internals/php7

- php 进阶探讨
  https://www.lanqiao.cn/library/advanced-php/

# 什麼是 CGI、FastCGI、PHP-FPM
CGI與FPM
CGI是一種協議，為了保證web server傳過來的資料是標準格式

比如說，如果請求 index.html，web server會去找到這個文件再丟給瀏覽器，但這個只限於靜態文件而已，如果是index.php呢，就需要去找php解析器來處理了，接下來web service就會把這個請求交給php 解析器處理，那會傳那一些資料呢?像是 post或是url還有http header等，CGI就是規定要傳哪些資料、以及怎麼樣的格式

FastCGI是什麼?
接下來又提到那FastCGI是什麼呢? FastCGI是用來提高CGI處理process性能用的

那FastCGI會怎麼實作呢?
當php啟動時、會去尋找php.ini、進行環境的初始化，如果不使用FastCGI的情況下，每一個請求都會做這個動作，很明顯浪費系統資源，所以FastCGI會先啟動一個master，解析配置文件用，接下來再啟動worker，當請求過來時、master會遞資訊給worker，然後接下來等下一個請求，有這個機制就不用每一次重新跑一次初始化的動作了

PHP-FPM
那PHP-FPM呢? Fastcgi是一個協議，其實是php-fpm實現了這個協議

php-fpm是管理fastcgi ，大概的關係就是這

所以最後才會造成如果修改php.ini檔案之後，才需要重新啟動php-fpm，原因就是這個樣子

https://github.com/UniSharp/blog/blob/master/2017/10/04/Nginx%20%E5%AD%B8%E7%BF%92%E7%AD%86%E8%A8%98%EF%BC%88%E4%BA%8C%EF%BC%89FastCGI%20Proxying/index.html
https://www.astralweb.com.tw/what-is-differences-between-fastcgi-php-fpm/
https://www.zybuluo.com/phper/note/50231

# php 請求外部資源

可以
file_get_contents
fopen
fsockopen
curl

# php 開啟服務

stream_socket_server()
workerman 的原理

# 重構

https://docs.google.com/presentation/d/10vxyi5VXSSZ4MMjLrKAIhrnpmuCjdDaZ_4K7ma1KIys/edit

## 避免用 switch

https://www.slideshare.net/kylinfish/php-best-practice-81744253
https://learnku.com/laravel/t/7468/please-keep-your-php-code-neat-and-tidy

# trait

官网讲的非常清楚
https://www.php.net/manual/zh/language.oop5.traits.php
唯一缺的范例是，当 trait 的 function 和要使用的 class 冲突时,你不想复盖，想要拿来用

```php
	use FooTrait {
		bar as traitBar;
	}
```

https://andy-carter.com/blog/overriding-extending-a-php-trait-method

# Reflection

获取档案路径 class function params 等等的详细资料
还可以用来实例一个物件,而非传统的 new 关键字做出实例

# fun fact

http_build_query()
https://stackoverflow.com/questions/14761418/browser-mis-interpreting-not-in-url

# array() vs []

版本問題，5.4 之後推薦使用[]為標準用法

# new static()

[PHP new self 、new static 比較](https://xyz.cinc.biz/2016/11/php-new-self-vs-new-static.html)
new self() => 生成的物件為實際寫有這句 code 的 class
new static() =>生成的物件為呼叫使用這句 code 的 class

# PHP_EOL

PHP_EOL 是一个些已经定义好的常量，代表 php 的换行符，这个变量会根据平台而变，在 windows 下会是/r/n，在 linux 下是/n，在 mac 下是/r。
$str = str_replace(PHP_EOL, '', $str);
這段程式碼，在單一來源的輸入沒問題，產出會隨著作業系統做變動，但是
如果
輸入有可能來自不同作業系統，我們沒辦法確定來源作業系統與 server 的相同
那會用到的情境是，我們有使用 rsa 密鑰，但他是.PEM 金鑰複製貼上的，為了避免 user 的 editor 太過聰明，直接依照作業系統換行，我們後端需要有處理換行的動作
https://www.cnblogs.com/cxx8181602/p/9132946.html

$str = str_replace(array("/r/n", "/r", "/n"), "", $str);
https://9iphp.com/web/php/1006.html

或是正則
$skuList = preg_split('/\r\n|\r|\n/', $\_POST['skuList']);

# php5.2

RSA256
https://stackoverflow.com/questions/10524198/what-version-of-openssl-is-needed-to-sign-with-sha256withrsaencryption

# php 雙引號

php 雙引號可以直接帶出變數

```php
echo "hello $name";
//等同於
echo "hello {$name}";
```

# php echo

```php
<?php echo $name; ?>
//等同於
<?= $name ?>
```

# php 防止跨站腳本

```php
htmlspecialchars($_Get['userInput']);
```

# php 節省 html 中的大括號{}

```php
foreach():
endforeach;
if():
endif;
```

# php compact

取變數名組成關聯陣列

# php extract

取關聯陣列 key value 產出變數

# php new self , new static

https://xyz.cinc.biz/2016/11/php-new-self-vs-new-static.html
new self() => 生成的物件為實際寫有這句 code 的 class
new static() =>生成的物件為呼叫使用這句 code 的 class
一般就推荐用 static() 才不会有机会实例出父类别

# 為什麼要用 require 而不是 include；為什麼要用 require 而不是 require_once

include 僅會拋出 Warning
require_once ，因為在執行時期完全沒有任何警告或錯誤，以致於造成冗餘程式碼
require 會在引入時發生錯誤時（例如被引入目標不存在）拋出 Error 且終止應用程式
composer autoload 有這 4 種方式
psr0,psr4,classmap,files 自動載入

# 製作 composer package

https://rivsen.github.io/post/how-to-publish-package-to-packagist-using-github-and-composer-step-by-step

# composer replace 取代一個套件

## fork 出沒有人維護的包,用 fork 的包來取代其他有用這個包的依賴

https://jaceju.net/composer-replace/

## 安裝沒有在 packagist 上註冊的私人套件

https://blog.johnsonlu.org/getting-started-with-composer/
https://www.hellosanta.com.tw/blog/how-to-add-3rd-party-plugin-by-using-composer

## 設定私有庫來源，並給予帳密

https://stackoverflow.com/questions/35082739/composer-asks-only-for-password-with-private-repository

# laravel 的方法

https://docs.chipperci.com/builds/composer-auth/

# composer autoload 的幾種方式

https://malagege.github.io/blog/2019/01/19/composer%E4%BD%BF%E7%94%A8psr0-psr4-classmap-files%E8%87%AA%E5%8B%95%E8%BC%89%E5%85%A5%E7%B4%80%E9%8C%84/

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

# 列出 php 加密演算法列表

print_r(hash_algos());

# phpinfo

commandline php -i

# xdebug 配置
[root] # cd /usr/local/src
[root] # tar zxvf xdebug-2.6.0RC2.tgz
[root] # cd xdebug-2.6.0RC2
[root] # /usr/local/php/bin/phpize
[root] # ./configure --enable-xdebug  --with-php-config=/usr/local/php/bin/php-config
[root] # make && make install
zend_extension=xdebug.so //指定Xdebug扩展文件的路径
xdebug.remote_enable=1 //是否开启远程调试
xdebug.remote_handler=dbgp //指定远程调试的处理协议
xdebug.remote_mode=req //可以设为req或jit，req表示脚本一开始运行就连接远程客户端，jit表示脚本出错时才连接远程客户端。
xdebug.remote_host=192.168.1.98 //指定远程调试的主机名（安装phpstorm的主机ip）
xdebug.remote_port=9001 //指定远程调试的端口号
xdebug.idekey="PHPSTORM" //指定传递给DBGp调试器处理程序的IDE Key
xdebug 配置並不簡單
https://segmentfault.com/a/1190000011907425
https://segmentfault.com/a/1190000018961750
https://www.yisu.com/zixun/39716.html


# disable_fucntion

https://www.anquanke.com/post/id/197745

# php lint

雖然 phpstorm 內建整合 php-cs-fixer ，跟 laravel 一樣的套件
https://laracasts.com/discuss/channels/laravel/php-cs-config-for-php-cs-fixer
阿我們還是希望協作者能夠在提交前跑一次格式檢查
此時就會用到
使用 Git pre-commit 自動修正 PHP 的 Coding Style
https://codefun.tw/2019/2019051901-php-coding-style-fix-with-git


# 字串比較

mbstring 使用了国家默认语言设置（NLS），所以在做字串長度比較時，需要注意
mbstring.language = UTF-8
mbstring.internal_encoding = UTF-8

# fastcgi_finish_request

当PHP运行在FastCGI模式时，PHP FPM提供了一个名为fastcgi_finish_request的方法。按照文档上的说法，此方法可以提高请求的处理速度，如果有些处理可以在页面生成完后再进行，就可以使用这个方法

https://blog.huoding.com/2011/04/12/63

# 精度問題

所有程式語言一定都有的問題，因為二進位無法完美表示浮點數
php官方建議如果需要高位數的計算的話，需要使用BC Math或是GMP
[關於php浮點數float以及int的問題](https://blog.walile.info/2013/06/14/about-php-float-problem/)

# PHP sleep() 是否会占用很多资源？

 sleep本身不占CPU资源, 但是在基于PHP的LAMP环境中, 就不能这样说了, 因为CPU不是唯一的资源. 进程数, 内存, 这些都是资源.
 
 # PHP運用多執行緒
 
 php可以利用其他的東西來實現偽多進程，多線程，例如：fsockopen實際是利用socket的多線程，popen，pcntl_fork ，proc_open利用httpd多進程功能的外衣
 [《面試官別再問》PHP運用多執行緒(Multi-thread)實現非阻塞方法](https://bps1025.blogspot.com/2018/10/php-multi-threadprocopen.html)
 [How do you make good use of multicore CPUs in your PHP/MySQL applications?](https://stackoverflow.com/questions/2267345/how-do-you-make-good-use-of-multicore-cpus-in-your-php-mysql-applications)

 # PHP+MySQL多语句执行
 http://www.jinglingshu.org/?p=3941
 实际上mysql早在4.1版本就允许多语句执行。只是PHP自身限制了这种用法。

 # composer autoload會佔據記憶體嗎
 答:不會

 如果下面連結說的是正確的話， 會在實際new的時候才載入CODE，記憶體一開始autoload只是一個檔案位址的陣列
 https://stackoverflow.com/questions/37283217/composer-dont-use-autoload-and-load-single-classes

 # 效能調校
 - where:local端測試
 - when: testing，deploying
 - what: 執行時間，記憶體使用量
 - 目標:在效能與可維護之間做平衡
 https://www.youtube.com/watch?v=hOajLLej68Y

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
 https://stackoverflow.com/questions/3255516/what-does-before-the-function-name-signify
 https://stackoverflow.com/questions/21058439/is-there-ever-a-need-to-use-ampersand-in-front-of-an-object

 # PHP 強制使用「強型態」的模式

 ```php
 declare(strict_types = 1);
 ```

 # json_encode() 浮点小数溢出错误 

 该现象只出现在PHP 7.1+版本上
 這個就是只有靠經驗才會知道的知識點
 可以靠serialize_precision這個解決

 # pdo和mysqli
 這兩個庫都是很熱門的資料庫API，各有優缺點，但pdo不只支援mysql

 [PDO vs. MySQLi: The Battle of PHP Database APIs](https://websitebeaver.com/php-pdo-vs-mysqli)

 # 假設你要測量一個php function的效能，要如何在不影響原有代碼的情況下，標記執行時間呢?

 https://stackoverflow.com/questions/9262158/how-can-i-inject-and-remove-php-code-before-a-function-call
 
 # json_deocde 

 在接收兩個幾乎相同的字串時，php json_decode 會報出 syntax error的錯
 錯的原因，雙引號前面多了個slash，奇怪的是我單純放進input後端magic quote產生有slash的字串不會錯，但是前端JSON.Stringfy()之後放進input 後端產生有slash的字串卻抱錯了
 https://stackoverflow.com/questions/14757983/json-post-with-magic-quotes-plus-a-quote-character-in-the-data
