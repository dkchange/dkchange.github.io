---
title: 安裝laravel
date: 2019-11-04 17:36:44
tags: laravel
categories:
  - [後端路線, framework, laravel]
---

# 安裝 laravel

本來想簡單用官網的 VirtualBox 搭配 Vagrant 來安裝(順帶一提 VirtualBox 沒有 command line 的模式，需要額外下載別的軟體下指令)，因為目前要搭的網站建在 AWS 的免費 EC2 方案，受限於電腦配備，虛擬機行不通、然後 docker 容器也行不通，所以認命重頭裝 QQ
BTW 簡中的 laravel 文檔，翻譯得比繁中的完整喔https://learnku.com/docs/laravel/5.7

http://kejyun.github.io/Laravel-5-Learning-Notes-Books/
https://github.com/oomusou/oomusou.github.io/issues/1

# 安裝過程

就是中間有報錯上網查，缺什麼裝什麼

    sudo apt-get update
    sudo apt install mariadb-server
    sudo apt install nginx

    再來就是php是命令行用的，php-fpm是server用的所以都要裝
    sudo apt-get install php7.2
    sudo apt-get install php7.2-fpm

    把laravel 需要的php擴展裝一裝
    sudo apt-get install php7.2-pdo
    sudo apt-get install php7.2-xml
    sudo apt-get install php7.2-mbstring

    像是連資料庫的時候會出現could not find driver就是缺
    sudo apt-get install php7.2-mysql


    下載composer執行檔，然後移到global的資料夾
    curl -sS https://getcomposer.org/installer | php
    mv composer.phar /usr/local/bin/composer

    composer self-update
    更新composer

    composer global require laravel/installer
    上面這行有報錯的話要給權限

    新增.bash_profile到自己的~目錄(/home/ubuntu/)下，然後加上
    export PATH=~/.composer/vendor/bin:$PATH
    儲存後登出
    這樣以ubuntu登入時就可以使用laravel這個指令
    cd /var/www/html/
    laravel new myproject

之後進`/etc/nginx/sites-enabled/default`

把 try_files 最後的 404 改成入口文件，[詳情請看](https://www.hi-linux.com/posts/53878.html)

    try_files $uri $uri/ /index.php?$query_string;
    設定nginx的網站根目錄
    root /var/www/html/myproject/public/;
    還有設定php7.2-fpm的位置
    fastcgi_pass unix:/run/php/php7.2-fpm.sock;

然後重啟 nginx

    systemctl restart nginx

之後進`/etc/nginx/nginx.conf`看看 nginx 的 user 是誰，改網站目錄

    chown -R www-data:www-data /var/www/html/myproject

# 最後

![成功](/images/成功.PNG)

# 資料庫設定

網路上教學清楚
https://gist.github.com/vicgonvt/cd0431a5cdc043ebab7f4954f7b4d471

# laravel 套件開發

http://kejyun.github.io/Laravel-5-Learning-Notes-Books/package/development/package-development-README.html


