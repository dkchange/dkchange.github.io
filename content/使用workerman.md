---
title: 使用workerman
date: 2020-09-17 12:29:35
tags:
categories:
---

# 前言

在許多的應用場合裡面，我們會需要定時執行或持續執行的 php，那實際應用方面，我們可能就會採用系統的排程來處理這件事，但是排程有幾個缺點
排程的時間是否能動態改變
排程如果在下一個排程之前都還沒執行完會不會造成資料錯誤
排程之間如果要如何共享資料
基於上述缺點
我們希望建一個 php 守護進程，如果要有一個 php 守護進程，我們必須要創一個無窮迴圈來處理這件事情
https://alexwebdevelop.com/php-daemons/
到最後出現
phpdaemon，reactphp，ratchet，workerman，swoole
等技術來處理我們的問題

直上官網連結
https://www.workerman.net/workerman
