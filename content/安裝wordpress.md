---
title: 安裝wordpress
date: 2020-07-09 14:51:26
tags:
categories:
- [wordpress,sage]
---

# 前言

對於接案的人來說，你重頭開始造輪子是不可能的，沒有那個時間精力，要不就是拿現有工作的 code 來改，要不就是開源專案選一個來寫，
那以前工作都是 laravel 客製化公司後台，對於 cms 這種案子的派不上太多用場，勢必要用流行的 cms 框架，其中就選擇最多人用的 wordpress
https://wordpress.org/showcase/

不過也出了純 js 版的專案
Github 平臺最有價值的 100 個專案
https://github.com/Automattic/wp-calypso

# wordpress 本地開發

網路上有很多資源介紹如何本地開發，和設定本地環境
https://roots.io/docs/bedrock/master/local-development/#additional-resources

# wordpress 網站範例

https://inciteresponse.com/hotel-websites-built-using-wordpress/

# 理想的 wordpress 開發流程

非常重要的影片
https://youtu.be/mzs-X2z-96Y

# oracle cloud

免費一年的 aws 過期了
出現 Permission denied (publickey).
一開始以為是預設金鑰抓錯了
ssh -i /home/stanley/.ssh/id_rsa ubuntu@168.138.46.17
後來才發現要指定使用者阿～～
ssh ubuntu@168.138.46.17

# trellis

看 github 的 about 介紹 Ansible playbooks for a WordPress

https://www.youtube.com/watch?v=-pOKTtAfJ8M
It appears your machine doesn't support NFS, or there is not an
adapter to enable NFS on this machine for Vagrant.

> sudo apt-get install portmap nfs-kernel-server

解釋一下 trellis 做了什麼，他幫忙在本機設定好虛擬的環境，就是有可能你是 windows 用戶 code 要放在 linux 上，但是他並不會在遠端也幫你建一個虛擬機，所以你的遠端伺服器的作業系統一定要是 ubuntu
Trellis runs on Ubuntu 18.04 LTS (Bionic Beaver) based servers

# 接案報價

阿接案的新手如果不知道報價的話，可以參考網路上的一些公司
https://wpw.design/price-wordpress/

# 開發

## wordpress 社群

wordpress 的社群氛圍相對其他框架，並不是那麼優良
因為 wordpress 定位為"不會寫程式的人也會用"，爾且全球使用的人實在太多，這造就了很多人問問題就只是要答案複製貼上，沒什麼反饋
我們都會這樣了，何況是不會寫程式的人，所以種種原因會讓你在問題定位上遇到困難

## wordpress 設計模式

義大利麵,wordpress 是很久的專案了，對於熟悉現代框架的我們，寫起來其實是相當痛苦，整個專案是程序導向的方式，沒什麼規範
https://wordpress.stackexchange.com/questions/169812/what-is-the-design-pattern-for-wordpress-core

那硬要說的話，wordpress 採事件驅動的開發模式，那缺點是什麼？
整個生命週期安插了許多的事件，使用者自訂的事件，你必須要對生命週期有一定程度的了解，在加上哪些地方有用的到這個事件，可能有很多地方都對這個事件添加了 handler
變成不好管理
hook function 就是我們俗稱的 callback
分為兩種
action hook 是 **沒有** 回傳值的 callback
filter hook 是 **有** 回傳值的 callback
然後如何決定這些 handler 的順序？
就會有個 Priority 參數
然後可以透過\$wp_filters['事件名']這個 global 變數來取得所有 hook 的資訊

https://audilu.com/2011/10/10/wordpress-hook/

然後 wordpress 在 php 還沒有物件導向時就出來了，也沒有硬性規定你的寫法，可以用程序導向，也可以用物件導向，所以在命名規則上，就會有一些約定，
像是 function 前面要加上主題的前綴，避免和插件等等命名發生重複

其餘內建的命名規則：
get_the_XXX()不會打印，只會回傳值
the_XXX()會印出東西

## 如何改進開發體驗

在開發 wordpress 時，會發現用到很多的 function 很多的 callback，所以選對 ide 來開發 wordpress 顯得相當重要，哪個時期哪些功能要用到哪些 wp 的 api 也很重要

wordpress 寫 code 容易寫出義大利麵
話雖然是這樣說，但裡面龐大的資源是大家不能夠輕易放棄的，所以就有許多 opensource 專案出來，希望讓 wordpress 開發比較符合現代開發方式

## 開發類型

https://developer.wordpress.org/

主要分成

- 插件開發
- 主題開發

你可以看到網路上許多在賣主題，或插件的，對我們接案的來說你的功能都會在主題開發完成，有需要共用的功能才需要花心力抽成插件

## sage 起手主題

sage 是能夠讓你快速開發 wordpress 主題的 Boilerplate，引入了 laravel 的 blade 模板引擎，以及前端工程化的 webpack
照著文檔規則走可以讓你很輕鬆的把現代開發融入 wordpress 主題開發之中，就是資料夾結構都幫你定義好了，照做就行

## 管理插件

wordpress 的插件都是放在自己定義的 plugins 資料夾，但我們現在都是用 composer 來管理現代專案了，所以必須要將原本指向 vendor 的資料夾改指向 plugins
https://n3xtchen.github.io/n3xtchen/php/2015/02/15/wordpress-n-composer
https://roots.io/using-composer-with-wordpress/

## bedrock

為了讓 wordpress 能夠使用 composere 管理插件，就出現了這個專案啦，但是這個專案並不能更新，因為他也是 Boilerplate,沒辦法像更新插件一樣輕鬆更新，就像你在更新框架也很麻煩一樣，不過沒關係 wordpress 可以正常更新就好
https://discourse.roots.io/t/how-to-update-wordpress-bedrock-after-deployment/3741/8

### 開放套件按裝和刪除

基於甲方可能亂刪套件亂裝套件導致網站壞掉，最好還是關掉這個功能
https://discourse.roots.io/t/does-bedrock-trellis-disable-the-ability-to-add-install-plugins-via-admin/6925/6

## wp-cli

wordpress 有成千上萬的套件，一個站一定會用上個幾個，那你又不可能每次都手動去後台安裝套件，再來啟用，太花時間了
就算插件能用 composer 做管理，插件的啟用還是得要上後台開啟，所以就有了 wp-cli 這工具讓你可以下指令做啟用，
只要你會寫腳本運用 wpcli，專案套件的管理和啟用就不是什麼難事，工程師就也不一定要用 composer 來做管理，wp—cli 能開啟關閉套件還更方便些

所以我想這也有可能是 wordpress 自今一直沒改資料夾結構的原因之一，旨在一般大眾就可以使用，平易近人，工程師自己要用 composer 就自己想辦法
https://wp-cli.org/zh-cn/

## 前後端分離

### wpapi

wordpress 設計的 rest api，讓你前後端分離，但是在設計上每個請求都會經過一大堆插件，不是直接與 db 做溝通，爾且也不是每個插件都支援 wpapi，要看作者有沒有寫

### wpgrahql

wordpress 就是好用在他的插件，當你使用 wpgrahql 前後端分離的時候，就需要考慮到說，那我用 wordpress 幹什麼？
好處是因為他是直接跟資料庫做溝通，不會經過插件，就變成你只需要他的後台

## ACF 自動導入 json

https://support.advancedcustomfields.com/forums/topic/auto-import-json-export-file/

## ACF 頁面編輯頁移除齒輪

https://support.advancedcustomfields.com/forums/topic/remove-edit-field-group-cog/

## CPTUI 自動導入 json

https://github.com/WebDevStudios/custom-post-type-ui/issues/381

## 安裝 wordpress

https://www.datanovia.com/en/lessons/using-docker-wordpress-cli-to-manage-wordpress-websites/
https://stackoverflow.com/questions/50999848/how-to-run-wp-cli-in-docker-compose-yml

https://medium.com/@tatemz/using-wp-cli-with-docker-21b0ab9fab79

https://gist.github.com/bradtraversy/faa8de544c62eef3f31de406982f1d42

要執行服務裡面的指令可以
sudo docker-compose run --rm wordpress-cli post list
docker-compose run --rm cli bash

alias wp="sudo docker-compose run --rm wordpress-cli"
這樣以後就可以
wp post list

# wordpress 翻譯

sudo apt-get install poedit
不這樣做的話，poedit 會沒有權限打開一些檔案

https://roots.io/docs/sage/9.x/localization/#generating-language-files

做本地化一直沒有成功，最後還是依賴上述連結所提的
See Sage_Polylang_Theme_Translation
相關插件來翻譯

# 不用 walker 做 menu

https://developer.wordpress.org/reference/functions/wp_get_nav_menu_items/

# 付費的 plugin 如何用 composer 安裝

https://roots.io/guides/private-or-commercial-wordpress-plugins-as-composer-dependencies/
https://roots.io/guides/acf-pro-as-a-composer-dependency-with-encrypted-license-key/
https://getcomposer.org/doc/articles/handling-private-packages-with-satis.md

# 如何一鍵搬移資料庫

https://github.com/jasperf/trellis-sync

https://roots.io/plugins/sync-script/

# 安裝 sage

會遇到問題

```console
.node-gyp/12.1.0/include/node/v8.h:3002:5: note: candidate constructor not viable: no known conversion from 'v8::Local<v8::Value>' to 'const v8::String::Utf8Value' for 1st argument
    Utf8Value(const Utf8Value&) = delete;
```

因為 node-sass needs to be version 4.12.0 for Node.js 12 support
所以記得去 update 一下 node-sass 的版本

# 如果發現套件沒安裝好，可能是你改完 composer.json 之後沒有 update

https://discourse.roots.io/t/trellis-deploy-composer-uses-old-version-of-package-loading-from-cache/7186

# 在佈署完之後裝啟用語言

https://discourse.roots.io/t/adding-wp-language-files-on-deploy/9212/34

# 在佈署完之後啟用插件

https://discourse.roots.io/t/activate-plugins-after-added-via-composer-json/12790
利用 composer 的 hook
但是，trellis 在 deploy 之後似乎不會每次都觸發 composer update 或 composer install
所以套件並沒有因此而啟用，這需要再測試一下
測試之後發現新加了 plugin 還是不會觸發 composer hook
看來在 trellis 要另尋方法了
https://discourse.roots.io/t/plugin-activation-task-during-vagrant-provisioning/5803/4
https://discourse.roots.io/t/composer-install-with-no-scripts-doesnt-work/13743/12

# 更自動化一點

Let’s say I would like every commit to github to be automatically deployed to my staging server?
不用在本機打指令，這樣就要用到 ci/cd 啟一個 server 來跑 deploy 指令了

https://discourse.roots.io/t/automated-deployment/6536

# 利用 ACF 客製化主題設定

https://youtu.be/Dlsr5uVOMRI

# 搭配 sage 要做 composer install 的 hook

https://discourse.roots.io/t/help-sage-autoloader-errors-after-activating-theme-after-seemingly-successful-deploy/9991

# 語言問題

https://discourse.roots.io/t/manage-plugin-translations-current-approaches/9108

https://github.com/roots/bedrock/issues/30

# 環境佈署

wp search-replace '//happypanda.subnet.vcn.oraclevcn.com/wp' '//happypanda.tw'

wp search-replace 'happypanda.subnet.vcn.oraclevcn.com/app/' 'happypanda.tw/wp-content/'

# 後續處理

安裝 lighthouse 來做網站相關優化處理

# 前端工程化

https://awdr74100.github.io/2020-02-26-webpack-cssloader-styleloader/

# trellis 備份相關腳本

等待研究
https://github.com/ItinerisLtd/trellis-backup-during-deploy

# wordpress api

wordpress 防 csrf
https://trepmal.com/2018/01/26/cookie-nonce-authentication-for-rest-api-curl-requests/

# 試著複寫插件提供的 fucntion

為了沒裝插件時也能正常執行

https://wordpress.stackexchange.com/questions/243168/providing-fallback-function-and-allow-override-by-plugin
或者應該把插件放到 must pulgin 才是

# 避免 query 錯

- wp_reset_query() - ensure that the main query has been reset to the original main query
- wp_reset_postdata() - ensures that the global \$post has been restored to the current post in the main query.

https://wordpress.stackexchange.com/questions/144343/wp-reset-postdata-or-wp-reset-query-after-a-custom-loop

# wp-cli

https://wp-cli.org/zh-cn/

# 從 bedrock 到正常的 wordpress

//url.example/wp -> //url.example
url.example/app/ -> url.example/wp-content/

docker-compose run --rm wordpress-cli search-replace '//happypanda.subnet.vcn.oraclevcn.com/wp' '//localhost:8000'

docker-compose run --rm wordpress-cli search-replace 'happypanda.subnet.vcn.oraclevcn.com/app/' 'localhost:8000/wp-content/'

然後 sage 裡面的 wp-content/themes/happy-panda/resources/assets/config.json
"publicPath": "/wp-content/themes/happy-panda"
要改 webpack 在編譯時路徑才會對

# poeidt

語言翻譯教學
https://www.youtube.com/watch?v=W5BkxT2dVo4
