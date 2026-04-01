---
title: laravel使用使用者認證
date: 2019-11-07 20:46:04
tags:
  - session
  - token
  - passport
categories:
  - 後端路線
  - framework
  - laravel
---

# 前言

[聊聊 Web API 認證方案那點事](https://mp.weixin.qq.com/s/2Zu4XVhomoPjM6mAj1dFIw)

前後端分離需要驗證，那怎樣讓會員可以不透過 session 維持登入狀態呢？
[[Laravel]] 內建有種驗證方式，可參考[簡聊 Session 與 Token 身份驗證](https://learnku.com/articles/17867)

- Session
- Token
  可以外掛
- [[JWT]] 可參考[API 開發中可選擇傳遞 token 介面遇到的一個坑](https://learnku.com/articles/13592/api-development-can-choose-to-pass-a-pit-encountered-by-token-interface)
- [[Passport]]
  Session 就不贅述，這邊主要介紹 API Token 和 Passport 的方式
  那官網一直到 Laravel 5.8 才有對 [API Authentication](https://laravel.com/docs/5.8/api-authentication) 的 Token 做出獨立的文件，
  之前有關於 API Authentication 的介紹都是 Passport，5.8 以後 Passport 才被移到官方套件欄位

# Token

## 內建 Token 缺什麼？

官方並沒有提供 API Token 的更新方式，也就是說，
以 Session 為例，你 web 用 Session 登出 Session ID 會失效，
那按照這道理，你 API 登出 API Token 也要失效，
否則你的 API 還是可以被訪問的，這樣大不如 API 也用 Session 來 guard？
要怎麼解決這問題？
用 JWT 或 Passport
或
實作登入登出刷新 Token 可參考[Laravel 自帶的 API 守衛驅動 Token 使用詳解](https://learnku.com/articles/11006/detailed-explanation-of-laravels-own-api-guard-drive-token)

# Passport

Laravel 的 API 驗證 API Authentication（Passport）官網推薦

    composer require laravel/passport
    php artisan passport:install --force

先把預設相關的表建到資料庫

    php artisan migrate

然後跟著官網

# 遭遇問題

Encryption keys already exist. Use the --force option to overwrite them.

# 支援 GraphQL

    composer require joselfonseca/lighthouse-graphql-passport-auth

Header 如果有多欄位會報錯（註解也不行），原因不明
例如在 Header 多加 Content-Type 已經註解起來了也不行
Syntax Error: Unexpected <EOF>

# 遭遇狀況

GuzzleHttp Hangs When Using Localhost
not localhost but multiple requests for a single-threaded web server are the problem. You still can use artisan, but you have to run another instance on another port e.g. with php artisan serve --port=8001 and tell Guzzle to use this base_uri (http://localhost:8001) instead
[Stack Overflow 討論](https://stackoverflow.com/questions/48841018/guzzle-cannot-make-get-request-to-the-localhost-port-80-8000-8080-etc/57573002#57573002)
[Stack Overflow 討論](https://stackoverflow.com/questions/36947844/guzzlehttp-hangs-when-using-localhost)

# 延伸閱讀-自定義認證

[基於 Laravel Auth 實現自定義介面 API 使用者認證詳解](https://learnku.com/articles/14136/realization-of-user-interface-authentication-for-user-interface-api-based-on-laravel-auth?order_by=vote_count&)

[Laravel API OAuth2 教學](https://jsnwork.kiiuo.com/archives/2937/laravel-api-oauth2-%E6%9E%B6%E8%A8%AD/)

# JWT VS Session

[不要用 JWT 替代 Session 管理（上）](https://zhuanlan.zhihu.com/p/38942172)（連結已失效）
