---
title: 使用vue
date: 2019-11-15 21:17:46
tags: vue
categories:
  - 前端路線
  - framework
  - vue
---

# 前言

之前學框架都是一點點碰，學都學半套，[[Angular]] 碰不到 2 個禮拜，[[React Native]] 碰不到 2 個月，學完就忘，現在來主動學 [[Vue]] 吧，畢竟是最早跟 [[Laravel]] 一起搭的前端框架

# 安裝

整個專案的建置可以看說明書慢慢裝，但是等真的裝好設定好就不知道牛年馬月，透過腳手架 [[Vue CLI]] 來裝會簡單些，等以後有機會再來了解如何配置

    sudo npm install -g @vue/cli

    vue create myvue

自選安裝的項目或者用高級的 GUI 來裝也行

Check the features needed for your project:
◉ Babel
◯ TypeScript
◯ Progressive Web App (PWA) Support
◉ Router
◉ Vuex
◯ CSS Pre-processors
◉ Linter / Formatter
◯ Unit Testing
◯ E2E Testing

難過的是即便用腳手架來裝問題還是會一堆

error Found incompatible module.
ERROR command failed: yarn --registry=https://registry.npm.taobao.org
user 底下.vuerc 改
{
"useTaobaoRegistry": false,
"packageManager": "npm",
}
[GitHub Issue](https://github.com/vuejs/vue-cli/issues/889#issuecomment-397225829)

[[Linux]] 環境還是失敗，[[Windows]] 環境倒是成功了，先在本機開發吧

# 安裝 Vue Apollo

    vue add apollo

error: 'vue-cli-plugin-apollo' should be listed in the project's dependencies, not devDependencies

    npm install vue-cli-plugin-apollo --save-prod

error: 'graphql-tag' should be listed in the project's dependencies, not devDependencies (import/no-extraneous-dependencies)

    npm cache clean --force

    npm install graphql-tag --save-prod

# 設置 IDE

可以發現專案在儲存時不會 autofix，
[Juejin 教學](https://juejin.im/post/5b39f274f265da59a07c5836)

```js
    "eslint.autoFixOnSave": true,
    "eslint.validate": [
        "javascript",
        "javascriptreact",
        {
            "language": "html",
            "autoFix": true
        },
        {
            "language": "vue",
            "autoFix": true
        }
    ],
```

# Vue 冷知識

你知道路徑裡的 @ 為什麼是 src 嗎？
[Stack Overflow 討論](https://stackoverflow.com/questions/42749973/es6-import-using-at-sign-in-path-in-a-vue-js-project-using-webpack)

```js
// @ is an alias to /src
import Home from "@/components/Home"
```

[GitHub Issue](https://github.com/vuejs/vue-apollo/issues/263)

Expected linebreaks to be 'LF' but found 'CRLF'.
最簡單的方法就是 [[VS Code]] 右下角切換成 LF

# GQL 檔案上傳

不用設定什麼就可以上傳檔案
遇到的問題：請求回傳
"message": "Undefined index: File",
"exception": "ErrorException",
"file": "/var/www/html/myproject/vendor/nuwave/lighthouse/src/Schema/TypeRegistry.php"
然後看 Header 那邊多了字串

    fragment file on File

真的是搞死我，gql 檔不要有空白或註解 ~~#import "./FileFragment.gql"~~ 刪掉就沒事了

```gql
#import "./FileFragment.gql"

mutation ($file: Upload!) {
  upload(file: $file)
}
```

# 測試

遭遇問題
Q:npm ERR! peer dep missing: canvas
A:Just upgrade your npm version to the latest.
https://github.com/vuejs/vue-cli/issues/4869

# 關於 yii 觸控事件

自己實作
九宮格相關
https://stackoverflow.com/questions/57723142/handling-touchstart-touchmove-touchend-the-same-way-as-mousedown-mousemove-mouse
長按事件
https://segmentfault.com/a/1190000013922322

相關的庫
