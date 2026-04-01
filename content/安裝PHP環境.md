---
title: 安裝PHP環境
date: 2019-12-21 20:22:50
tags:
categories:
---

# 前言

事情是這樣子的，以前從 Windows 環境就安裝 [[AppServ]]、[[WampServer]]、[[XAMPP]] 這種快速安裝的安裝包，為什麼呢？
因為最後環境還是會在 Linux 上，所以 Windows 上能開發就好，也就不用太講究了，因為有些套件在 Linux 沒問題，在 Windows 會出事，
心血來潮試著自己安裝一次看看

# 名詞解釋

## X86 / X32 / X64

我們要安裝 [[PHP]] 時就會看到一些不懂的名詞

- X86 = X32 = 32 位元
- X64 = 64 位元
  為何這樣奇怪的命名就是歷史因素了

## Thread Safe vs None Thread Safe

[線程安全問題](https://www.awaimai.com/88.html)，這就牽扯到比較複雜的因素了，有時間再看。
因為我們是開發機，先不考慮線程安全和效能，任選一種皆可。

# PHP-FPM 時區設定

如果你是跑 [[php-fpm]] 那設定在 `/etc/php.ini` 的時區你會吃不到，此時就要去
`/etc/php-fpm.d/www.conf` 加上：

```conf
php_admin_value[date.timezone] = Europe/Berlin
```

# 遭遇問題 - XAMPP MySQL 啟動失敗

今天打開 XAMPP 試著開啟 MySQL 服務的時候，程式出錯：

> Error: MySQL shutdown unexpectedly.

打開日誌看不出個什麼鬼，只知道突然中斷，日誌都是正常的紀錄，然後就開始爬文。

嘗試過的方法：

1. 在 `mysql/bin/my.ini` 文件加一個：

   ```
   [mysqld]
   innodb_force_recovery = 4
   ```

   → 失敗

2. 把 `xampp\mysql\data` 下的 `ibdata1` 文件刪掉
   → 失敗

3. 用管理員啟動命令行，進入 MySQL 的 bin 目錄，輸入 `mysqld --install`，`net start mysql`
   → 失敗

4. 將 `ib_logfile0` 以及 `ib_logfile1` 這兩個日誌檔刪除
   → 失敗

5. 把 `xampp/mysql/backup/` 底下所有檔案，複製取代到 `xampp/mysql/data/`
   → **成功！**

   參考：[DotBlogs 教學](https://dotblogs.com.tw/mepowerlmay/2019/12/19/085908)

# 常見錯誤

## 403 Forbidden

除了在 [[Nginx]] 有設定以外，如果本身資料夾權限不足，nginx 的進程使用者也會報錯

## No input file specified

找不到檔案，很可能就是 Nginx 的正則寫錯了

## 502 Bad Gateway

[CentOS 教學](https://www.centos.bz/2017/07/nginx-php-fpm-502-error/)、[ServerFault 討論](https://serverfault.com/questions/457911/nginx-php-fpm-502-bad-gateway)
發生這個問題的原因有很多，之前還有可能是 CDN 快取導致頁面一直出現這個錯誤。

# Xdebug 環境配置

[SegmentFault 教學](https://segmentfault.com/a/1190000011907425)

Ubuntu 安裝 PHP Xdebug：

```bash
sudo apt-get install php-xdebug
sudo phpenmod xdebug
sudo gedit /etc/php/7.3/mods-available/xdebug.ini
# 重啟 webserver
```

[Debugging PHP on Linux with Xdebug and PHPStorm](https://youtu.be/3idASlzGTg4) 介紹三種方式：

1. 在瀏覽器 URL 打上參數
2. 用 PHPStorm 網站提供的書籤
3. 安裝 Chrome 擴充套件
   - [Stack Overflow 討論](https://stackoverflow.com/questions/46263043/how-to-setup-docker-phpstorm-xdebug-on-ubuntu-16-04)
   - [YouTube 教學](https://www.youtube.com/watch?v=mahIIF0c8Zo)
   - 使用 `xdebug.remote_host=host.docker.internal` 比硬編 IP 位址好
   - 但 Linux 版本要看有沒有支援
   - `xdebug.remote_host=docker.for.mac.localhost` 是 Mac 的配置方式
   - Docker on Linux 可以直接設定 `xdebug.remote_connect_back=1`，不需要指定 remote_host

Linux 不支援 host.docker.internal，可參考 [Dev.to 文章](https://dev.to/bufferings/access-host-from-a-docker-container-4099)。

# Nginx 設定

[Nginx 設定教學](https://blog.51cto.com/13930997/2311716)

[Regex101](https://regex101.com/) - 正則表達式線上測試工具

## Domain Name

如果你的測試機是以 `xxx.localhost` 來當 domain 的話，那你的 `hosts` 檔就可以不用改了，因為他自己會幫你導向本機

## index.php failed (13: Permission denied)

關閉 [[SELinux]] 即可：

```bash
setenforce 0
```

# Apache 設定

## Virtual Host

用 `site-available` 來設定 vhost，`site-enabled` 是 Apache 自己產生的，不要手動去改他：

```bash
sudo a2ensite vhost_domain_name
# site-enabled 就會產生出對應的檔案了
```

# Apache2 conf 設定檔中文版

[LinuxCoolTea 教學](https://sites.google.com/site/linuxcooltea/apache2/a) 這篇詳細解釋 conf 內容，對於我們這種非網管人員幫助很大
