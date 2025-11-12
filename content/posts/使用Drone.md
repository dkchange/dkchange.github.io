---
title: 使用Drone
date: 2020-01-20 14:11:20
tags:
  - travisCI
categories:
  - [DevOps路線, CI/CD工具, Drone]
---

# 前言

對於現代化的開發來說，你想要達到的自動化是什麼？最好是我 code 提交上去之後，一切都成功的佈署好，中間的過程有
代碼提交
代碼測試
代碼審核
代碼合併
代碼佈署
環境建制
代碼編譯

那這中間的過程，如果沒有導入 ci\cd 我們原本是怎麼做的？
git 提交
ftp 手動上傳 10 台？
代碼測試
每一台環境手動重新安裝？
在不同環境下不同指令編譯
寫許多腳本來環境安裝

太多名詞在網路上飛， 基於**沒用過至少也要聽過**，為了跟得上時代，有些東西還是免不了，這篇整理自
[就是「懶」才更需要重視 DevOps](https://ithelp.ithome.com.tw/users/20120491/ironman/2538)
[從零開始建立自動化發佈的流水線](https://ithelp.ithome.com.tw/users/20107551/ironman/1906)
[用 Drone 打造 CI/CD flow](https://medium.com/asiayo-engineering/%E7%94%A8-drone-%E6%89%93%E9%80%A0-ci-cd-flow-36b9d14c7620)
的讀後感心得，特別推 **從零開始建立自動化發佈的流水線**

# play with docker

https://labs.play-with-docker.com/
可以免費玩 docker 4hr 的服務

# 何謂 DevOps

**開發** 與 **維運** 融會貫通後的一種技術能力體現
只是，時間終究有限，需要 「自動化」 協助「DevOps」人員
是現代敏捷開發所秉持的精神

# 版本控制系統( Version Control System, VCS)

大致上分為兩種類型，集中式與分散式
代表案例：
Subversion 就是集中式版本控制系統
**Git**、Mercurial 則是分散式版本控制系統。

品牌認識:GitHub、BitBucket、Azure DevOps、GitLab

# git 開發流程（工作流程）

團隊合作如何避免衝突、覆蓋，分支如何合併等等的工作流程。
團隊間可以自行協調出工作流程或是根據知名的規範走
規範認識:git flow、Github flow、Gitlab flow

# 何謂 CI/CD

CI(Continuous integration)，持續整合，利用頻繁地**提交**新功能的變更，觸發自動化建置和測試
CD(Continuous Deployment)，持續佈署，快速而自動的**部署**，任何版本的軟體到不同的環境
CD(Continuous Delivery)，持續交付，儘快將最新版本的軟體，交付到最終使用者(end-user)手中

## CI server

可以自己架，或者使用線上平台提供架好的 CI 應用程式，這可以用來整合其他應用
知名品牌:Drone ,Jenkins, travisCI ,GitLab CI, CircleCI ,github action
以 drone 為例，其流程大致為：
clone git repo
run linter
run unit test
prepare bulid
publish docker
deploy
nofity

github ations https://www.youtube.com/watch?v=kuRhHaumEtE

## 自動訊息通知

CI server 發 mail 或與現有的知名通訊軟體做整合，或寫 API

## 自動單元測試

被測試的物件，商業邏輯的是否正確、輸出入的結果是否符合預期
單元測試的運用與觀念，非常重視 物件導向的特性 與 SOLID 原則。所以，假若不常用或不熟悉物件導向的開發者，初期不容易切入單元測試的世界。
單元測試，真正發揮它的 POWER 的時間點，在於軟體後續長期維護、需求變動、重構時，才能真正的明白為什麼需要單元測試。
自己寫腳本

## 自動部屬

自己寫腳本與現有的知名雲端平台做整合

## 部屬完的環境設定

如果每次部屬一台機器就要設定一次是很煩的事，服
所以就有了容器這項技術
直接針對 應用程式層下手，將 應用程式本身與相關的函式庫、環境配置檔，打包成映像檔 (Image)
知名專案:Docker

## 容器的集群管理平台

服務編排的功能
可讓您執行 Docker 容器和工作負載，並協助您解決在調整跨多部伺服器部署之多個容器規模時的一些複雜作業
知名專案:Mesos,Docker Swarm,K8S
