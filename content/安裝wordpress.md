---
title: 安裝wordpress
date: 2020-07-09 14:51:26
tags:
categories:
  - WordPress
  - Sage
---

# 前言

對於接案的人來說，從頭開始造輪子是不可能的，沒有那個時間精力。要不就是拿現有工作的 code 來改，要不就是從開源專案選一個來寫。
那以前工作都是 [[Laravel]] 客製化公司後台，對於 CMS 這種案子的派不上太多用場，勢必要用流行的 CMS 框架，其中就選擇最多人用的 [[WordPress]]。

不過也有純 JS 版的專案 - [wp-calypso](https://github.com/Automattic/wp-calypso)（GitHub 平台上最有價值的 100 個專案之一）。

# WordPress 本地開發

網路上有很多資源介紹如何本地開發和設定本地環境，可參考 [Roots.io Bedrock 文件](https://roots.io/docs/bedrock/master/local-development/#additional-resources)。

# WordPress 網站範例

[使用 WordPress 建的飯店網站](https://inciteresponse.com/hotel-websites-built-using-wordpress/)

# 理想的 WordPress 開發流程

非常重要的影片：[YouTube 教學](https://youtu.be/mzs-X2z-96Y)

# Oracle Cloud

免費一年的 AWS 過期了，出現 `Permission denied (publickey)`。一開始以為是預設金鑰抓錯了：

```bash
ssh -i /home/stanley/.ssh/id_rsa ubuntu@168.138.46.17
```

後來才發現要指定使用者阿～～

```bash
ssh ubuntu@168.138.46.17
```

# Trellis

看 GitHub 的 about 介紹：Ansible playbooks for a WordPress。

> 參考影片：[YouTube 教學](https://www.youtube.com/watch?v=-pOKTtAfJ8M)

遇到問題：

> It appears your machine doesn't support NFS, or there is not an adapter to enable NFS on this machine for Vagrant.

解決方法：

```bash
sudo apt-get install portmap nfs-kernel-server
```

解釋一下 Trellis 做了什麼：他幫忙在本機設定好虛擬的環境，就是有可能你是 Windows 用戶 code 要放在 Linux 上，但是他並不會在遠端也幫你建一個虛擬機，所以你的遠端伺服器的作業系統一定要是 Ubuntu。

> Trellis runs on Ubuntu 18.04 LTS (Bionic Beaver) based servers

# 接案報價

阿接案的新手如果不知道報價的話，可以參考 [WPWebDesign 報價](https://wpw.design/price-wordpress/)。

# 開發

## WordPress 社群

WordPress 的社群氛圍相對其他框架並不是那麼優良。因為 WordPress 定位為「不會寫程式的人也會用」，而且全球使用的人實在太多，這造就了很多人問問題就只是要答案複製貼上，沒什麼反饋。我們都會這樣了，何況是不會寫程式的人，所以種種原因會讓你在問題定位上遇到困難。

## WordPress 設計模式

義大利麵。WordPress 是很久的專案了，對於熟悉現代框架的我們，寫起來其實是相當痛苦，整個專案是程序導向的方式，沒什麼規範。可參考 [WordPress Stack Exchange 討論](https://wordpress.stackexchange.com/questions/169812/what-is-the-design-pattern-for-wordpress-core)。

那硬要說的話，WordPress 採**事件驅動**的開發模式。那缺點是什麼？

整個生命週期安插了許多的事件，使用者自訂的事件，你必須要對生命週期有一定程度的了解，再加 上哪些地方有用到這個事件，可能有很多地方都對這個事件添加了 handler，變成不好管理。

**Hook function** 就是我們俗稱的 callback，分為兩種：

- **Action hook**：沒有回傳值的 callback
- **Filter hook**：有回傳值的 callback

然後如何決定這些 handler 的順序？會有個 Priority 參數。接著可以透過 `$wp_filters['事件名']` 這個 global 變數來取得所有 hook 的資訊。

> 參考：[WordPress Hook 詳解](https://audilu.com/2011/10/10/wordpress-hook/)

然後 WordPress 在 PHP 還沒有物件導向時就出來了，也沒有硬性規定你的寫法，可以用程序導向，也可以用物件導向，所以在命名規則上就會有一些約定，像是 function 前面要加上主題的前綴，避免和插件等等命名發生重複。

其餘內建的命名規則：

- `get_the_XXX()` 不會打印，只會回傳值
- `the_XXX()` 會印出東西

## 如何改進開發體驗

在開發 WordPress 時，會發現用到很多的 function 很多的 callback，所以選對 IDE 來開發 WordPress 顯得相當重要，哪個時期哪些功能要用到哪些 WP 的 API 也很重要。

WordPress 寫 code 容易寫出義大利麵。話雖然是這樣說，但裡面龐大的資源是大家不能夠輕易放棄的，所以就有許多 open source 專案出來，希望讓 WordPress 開發比較符合現代開發方式。

## 開發類型

[WordPress 官方開發文件](https://developer.wordpress.org/)

主要分成：

- 插件開發
- 主題開發

你可以看到網路上許多在賣主題或插件的，對我們接案的來說你的功能都會在主題開發完成，有需要共用的功能才需要花心力抽成插件。

## Sage 起手主題

[[Sage]] 是能夠讓你快速開發 WordPress 主題的 Boilerplate，引入了 [[Laravel]] 的 [[Blade]] 模板引擎，以及前端工程化的 [[Webpack]]。照著文檔規則走可以讓你很輕鬆的把現代開發融入 WordPress 主題開發之中，就是資料夾結構都幫你定義好了，照做就行。

## 管理插件

WordPress 的插件都是放在自己定義的 plugins 資料夾，但我們現在都是用 [[Composer]] 來管理現代專案了，所以必須要將原本指向 vendor 的資料夾改指向 plugins。

## Bedrock

為了讓 WordPress 能夠使用 [[Composer]] 管理插件，就出現了 [[Bedrock]] 這個專案啦

## WP-CLI

[[WP-CLI]] 是 WordPress 的命令行工具，可以讓你用指令管理套件。

## ACF 自動導入 JSON

[[ACF]]

> 參考：[ACF 官方論壇](https://support.advancedcustomfields.com/forums/topic/auto-import-json-export-file/)

## ACF 頁面編輯頁移除齒輪

> 參考：[ACF 官方論壇](https://support.advancedcustomfields.com/forums/topic/remove-edit-field-group-cog/)

## CPTUI 自動導入 JSON

> 參考：[GitHub Issues](https://github.com/WebDevStudios/custom-post-type-ui/issues/381)

## 安裝 WordPress

> 參考：
>
> - [Datanovia 教學](https://www.datanovia.com/en/lessons/using-docker-wordpress-cli-to-manage-wordpress-websites/)
> - [Stack Overflow 討論](https://stackoverflow.com/questions/50999848/how-to-run-wp-cli-in-docker-compose-yml)
> - [Medium 文章](https://medium.com/@tatemz/using-wp-cli-with-docker-21b0ab9fab79)
> - [Gist 範例](https://gist.github.com/bradtraversy/faa8de544c62eef3f31de406982f1d42)

要執行服務裡面的指令可以：

```bash
sudo docker-compose run --rm wordpress-cli post list
docker-compose run --rm cli bash
```

設定 alias：

```bash
alias wp="sudo docker-compose run --rm wordpress-cli"
```

這樣以後就可以：

```bash
wp post list
```

# WordPress 翻譯

```bash
sudo apt-get install poedit
```

不這樣做的話，Poedit 會沒有權限打開一些檔案。

> 參考：[Roots.io Sage 本地化文件](https://roots.io/docs/sage/9.x/localization/#generating-language-files)

做本地化一直沒有成功，最後還是依賴上述連結所提的
See Sage_Polylang_Theme_Translation
相關插件來翻譯

# 不用 Walker 做 Menu

> 參考：[WordPress 官方文件](https://developer.wordpress.org/reference/functions/wp_get_nav_menu_items/)

# 付費的 Plugin 如何用 Composer 安裝

> 參考：
>
> - [Roots.io 指南](https://roots.io/guides/private-or-commercial-wordpress-plugins-as-composer-dependencies/)
> - [Roots.io ACF Pro 指南](https://roots.io/guides/acf-pro-as-a-composer-dependency-with-encrypted-license-key/)
> - [Composer 私有套件教學](https://getcomposer.org/doc/articles/handling-private-packages-with-satis.md)

# 如何一鍵搬移資料庫

> 參考：
>
> - [trellis-sync GitHub](https://github.com/jasperf/trellis-sync)
> - [Roots.io Sync Script](https://roots.io/plugins/sync-script/)

# 安裝 Sage

會遇到問題：

```
.node-gyp/12.1.0/include/node/v8.h:3002:5: note: candidate constructor not viable: no known conversion from 'v8::Local<v8::Value>' to 'const v8::String::Utf8Value' for 1st argument
    Utf8Value(const Utf8Value&) = delete;
```

因為 node-sass needs to be version 4.12.0 for Node.js 12 support，所以記得去 update 一下 node-sass 的版本。

# 如果發現套件沒安裝好，可能是你改完 composer.json 之後沒有 update

> 參考：[Roots Discourse 討論](https://discourse.roots.io/t/trellis-deploy-composer-uses-old-version-of-package-loading-from-cache/7186)

# 在佈署完之後安裝啟用語言

> 參考：[Roots Discourse 討論](https://discourse.roots.io/t/adding-wp-language-files-on-deploy/9212/34)

# 在佈署完之後啟用插件

> 參考：[Roots Discourse 討論](https://discourse.roots.io/t/activate-plugins-after-added-via-composer-json/12790)

利用 Composer 的 hook，但是 Trellis 在 deploy 之後似乎不會每次都觸發 composer update 或 composer install，所以套件並沒有因此而啟用，這需要再測試一下。測試之後發現新加了 plugin 還是不會觸發 composer hook，看來在 Trellis 要另尋方法了。

> 參考：
>
> - [Roots Discourse 討論](https://discourse.roots.io/t/plugin-activation-task-during-vagrant-provisioning/5803/4)
> - [Roots Discourse 討論](https://discourse.roots.io/t/composer-install-with-no-scripts-doesnt-work/13743/12)

# 更自動化一點

> Let’s say I would like every commit to github to be automatically deployed to my staging server?

不用在本機打指令，這樣就要用到 CI/CD 啟一個 server 來跑 deploy 指令了。

> 參考：[Roots Discourse 討論](https://discourse.roots.io/t/automated-deployment/6536)

# 利用 ACF 客製化主題設定

> 參考：[YouTube 教學](https://youtu.be/Dlsr5uVOMRI)

# 搭配 Sage 要做 Composer Install 的 Hook

> 參考：[Roots Discourse 討論](https://discourse.roots.io/t/help-sage-autoloader-errors-after-activating-theme-after-seemingly-successful-deploy/9991)

# 語言問題

> 參考：
>
> - [Roots Discourse 討論](https://discourse.roots.io/t/manage-plugin-translations-current-approaches/9108)
> - [Bedrock GitHub Issue](https://github.com/roots/bedrock/issues/30)

# 環境佈署

```bash
wp search-replace '//happypanda.subnet.vcn.oraclevcn.com/wp' '//happypanda.tw'
wp search-replace 'happypanda.subnet.vcn.oraclevcn.com/app/' 'happypanda.tw/wp-content/'
```

# 後續處理

安裝 Lighthouse 來做網站相關優化處理。

# 前端工程化

> 參考：[awdr74100 教學](https://awdr74100.github.io/2020-02-26-webpack-cssloader-styleloader/)

# Trellis 備份相關腳本

等待研究：[GitHub 專案](https://github.com/ItinerisLtd/trellis-backup-during-deploy)

# WordPress API

WordPress 防 CSRF：

> 參考：[trepmal 文章](https://trepmal.com/2018/01/26/cookie-nonce-authentication-for-rest-api-curl-requests/)

# 試著覆寫插件提供的 Function

為了沒裝插件時也能正常執行：

> 參考：[WordPress Stack Exchange](https://wordpress.stackexchange.com/questions/243168/providing-fallback-function-and-allow-override-by-plugin)

或者應該把插件放到 Must Use Plugin 才是。

# 避免 Query 錯

- `wp_reset_query()` - ensure that the main query has been reset to the original main query
- `wp_reset_postdata()` - ensures that the global $post has been restored to the current post in the main query.

> 參考：[WordPress Stack Exchange](https://wordpress.stackexchange.com/questions/144343/wp-reset-postdata-or-wp-reset-query-after-a-custom-loop)

# wp-cli

[[WP-CLI]] 官網：https://wp-cli.org/zh-cn/

# 從 bedrock 到正常的 wordpress

//url.example/wp -> //url.example
url.example/app/ -> url.example/wp-content/

docker-compose run --rm wordpress-cli search-replace '//happypanda.subnet.vcn.oraclevcn.com/wp' '//localhost:8000'

docker-compose run --rm wordpress-cli search-replace 'happypanda.subnet.vcn.oraclevcn.com/app/' 'localhost:8000/wp-content/'

然後 sage 裡面的 wp-content/themes/happy-panda/resources/assets/config.json
"publicPath": "/wp-content/themes/happy-panda"
要改 webpack 在編譯時路徑才會對

# Poedit

語言翻譯教學
[YouTube 教學](https://www.youtube.com/watch?v=W5BkxT2dVo4)
