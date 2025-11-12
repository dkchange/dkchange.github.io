---
title: laravel使用使用者認證
date: 2019-11-07 20:46:04
tags:
- session
- token
- passport
categories:
- [後端路線,framework,laravel]
---
# 前言
[聊聊 Web API 认证方案那点儿事](https://mp.weixin.qq.com/s/2Zu4XVhomoPjM6mAj1dFIw)

前後端分離需要驗證，那怎樣讓會員可以不透過session維持登入狀態呢?
laravel 內建有種驗證方式，可參考[简聊 Session 与 Token 身份验证](https://learnku.com/articles/17867)
- seesion
- token
可以外掛
- jwt 可參考[API 开发中可选择传递 token 接口遇到的一个坑](https://learnku.com/articles/13592/api-development-can-choose-to-pass-a-pit-encountered-by-token-interface)
- passport
session就不贅述，這邊主要介紹api_token和passport的方式
那官網一直到laravel 5.8 才有對[api API Authentication](https://laravel.com/docs/5.8/api-authentication) 的token做出獨立的文件，
之前有關於API Authentication的介紹都是passport，5.8以後passport才被移到官方套件欄位
# token
## 內建token缺什麼?
官方並沒有提供api_token的更新方式，也就是說，
以session為例，你web用session登出sessionID會失效，
那按照這道理，你api登出api_token也要失效，
否則你的api還是可以被訪問的，這樣大不如api也用session來guard?
要怎解決這問題?
用jwt或passport
或
實作登入登出刷新token 可參考[Laravel 自带的 API 守卫驱动 token 使用详解](https://learnku.com/articles/11006/detailed-explanation-of-laravels-own-api-guard-drive-token)

# passport


laravel 的API驗證API Authentication(passport)官網推薦
 
    composer require laravel/passport
    php artisan passport:install --force

先把預設相關的表建到資料庫

    php artisan migrate

然後跟著官網

# 遭遇問題
Encryption keys already exist. Use the --force option to overwrite them.

# 支援graphQL

    composer require joselfonseca/lighthouse-graphql-passport-auth

header如果有多欄位會報錯(註解不也不行)，原因不明
例如在header多加//Content-Type已經註解起來了也不行
Syntax Error: Unexpected <EOF>

# 遭遇狀況
GuzzleHttp Hangs When Using Localhost
not localhost but multiple requests for a single-threaded web server are the problem. You still can use artisan, but you have to run another instance on another port e.g. with php artisan serve --port=8001 and tell Guzzle to use this base_uri (http://localhost:8001) instead
https://stackoverflow.com/questions/48841018/guzzle-cannot-make-get-request-to-the-localhost-port-80-8000-8080-etc/57573002#57573002
https://stackoverflow.com/questions/36947844/guzzlehttp-hangs-when-using-localhost

# 延伸閱讀-自定義認證
[基于 Laravel Auth 实现自定义接口 API 用户认证详解]https://learnku.com/articles/14136/realization-of-user-interface-authentication-for-user-interface-api-based-on-laravel-auth?order_by=vote_count&

https://jsnwork.kiiuo.com/archives/2937/laravel-api-oauth2-%E6%9E%B6%E8%A8%AD/

# JWT VS Session

[不要用JWT替代session管理（上）](https://zhuanlan.zhihu.com/p/38942172)



