---
title: 安裝laravel
date: 2019-11-04 17:36:44
tags: laravel
categories:
  - 後端路線
  - framework
  - laravel
---

# 安裝 Laravel

本來想簡單用官網的 [[VirtualBox]] 搭配 [[Vagrant]] 來安裝（順帶一提 VirtualBox 沒有 command line 的模式，需要額外下載別的軟體下指令），因為目前要搭的網站建在 [[AWS]] 的免費 EC2 方案，受限於電腦配備，虛擬機行不通、然後 [[Docker]] 容器也行不通，所以認命重頭裝 QQ

> BTW：簡中的 [[Laravel]] 文檔，翻譯得比繁中的完整喔！[Laravel 5.7 文檔](https://learnku.com/docs/laravel/5.7)

參考資源：

- [KeJyun Laravel 學習筆記](http://kejyun.github.io/Laravel-5-Learning-Notes-Books/)
- [oomusou Laravel 教學](https://github.com/oomusou/oomusou.github.io/issues/1)

# 安裝過程

就是中間有報錯上網查，缺什麼裝什麼：

```bash
# 更新套件
sudo apt-get update

# 安裝資料庫
sudo apt install mariadb-server

# 安裝 Web Server
sudo apt install nginx

# PHP 命令行和 php-fpm 伺服器都要裝
sudo apt-get install php7.2
sudo apt-get install php7.2-fpm

# Laravel 需要的 PHP 擴充
sudo apt-get install php7.2-pdo
sudo apt-get install php7.2-xml
sudo apt-get install php7.2-mbstring

# 連資料庫時出現 "could not find driver" 就是缺這個
sudo apt-get install php7.2-mysql

# 下載 [[Composer]] 並移到 global 目錄
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

# 更新 Composer
composer self-update

# 安裝 Laravel installer
composer global require laravel/installer
# 上面這行有報錯的話要給權限

# 新增 .bash_profile 到自己家目錄 (/home/ubuntu/)
export PATH=~/.composer/vendor/bin:$PATH
# 儲存後登出，這樣以 ubuntu 登入時就可以使用 laravel 指令

# 建立新專案
cd /var/www/html/
laravel new myproject
```

之後編輯 `/etc/nginx/sites-enabled/default`，把 `try_files` 最後的 404 改成入口文件，[詳情請看 HiLinux 教學](https://www.hi-linux.com/posts/53878.html)：

```nginx
try_files $uri $uri/ /index.php?$query_string;
# 設定 [[Nginx]] 的網站根目錄
root /var/www/html/myproject/public/;
# 設定 php7.2-fpm 的位置
fastcgi_pass unix:/run/php/php7.2-fpm.sock;
```

然後重啟 Nginx：

```bash
sudo systemctl restart nginx
```

之後進 `/etc/nginx/nginx.conf` 看看 Nginx 的 user 是誰，改網站目錄權限：

```bash
sudo chown -R www-data:www-data /var/www/html/myproject
```

# 最後

![成功](/images/成功.PNG)

# 資料庫設定

[Laravel 資料庫設定 gist](https://gist.github.com/vicgonvt/cd0431a5cdc043ebab7f4954f7b4d471)

# Laravel 套件開發

[KeJyun Laravel 套件開發文檔](http://kejyun.github.io/Laravel-5-Learning-Notes-Books/package/development/package-development-README.html)
