---
title: 使用webRTC
date: 2020-11-13 12:41:05
tags:
categories:
---

# 參考資料

[WebRTC Crash Course](https://youtu.be/FExZvpVvYxA)
[30 天之即時網路影音開發攻略](https://ithelp.ithome.com.tw/users/20089358/ironman/1923)

# 前言

webrtc(web realtime communication) 可以讓兩個瀏覽器之間 p2p 直接溝通，而中間不需要在透過 server，達到的目的就是快速

# sdp(sesssion description protocol)

在兩個瀏覽器溝通之前，必須先傳給對方用 webrtc 溝通所需資料，這個過程我們稱作 signaling，而這些資料我們稱為 sdp

signaling 這個動作可以藉由任何管道，可以已藉由通訊軟體，websocket 只要有辦法傳送給對方即可，那通常大家都會採用 websocket 來建一個 signaling server

sdp 裡面會包含很多的資料，傳輸的格式，加密的方法，ice candidates 好多好多資料

A → B A 把 offer(SDP)給 B
B ← A B 收到之後，用這個 SDP 在創建一個(SDP)answer 給 B

# nat(network address translation)

如果你有 public ip 一切都變得簡單，但大部分的人都沒有，那我們就需要透過 nat，nat 簡單來說就是一張表，當你要和外部溝通時，他會建立對應的表
，那收到回應時，我們就會來查這張表
你的 ip | 你的 port | router ip | router port | host ip | host port
那不同的 router 有不同的機制

- one to one nat(full-cone nat) 不管你之前有沒有訪問過這 ip 都允許他和你溝通
- address restricted nat 只要你之前有有訪問過這 ip，就允許他和你溝通
- port restricted nat 只要你之前有有訪問過這 ip 爾且 port 是對的，就允許他和你溝通
- symmetric nat 比必須要 router 出去的 port 和 ip，訪問的主機 ip 和 port 這四個東西都相同，我才允許你回應

然而 webrtc 跟 symmetric nat 相較之下比較沒那麼相容，因為 webrtc 需要有一個 stun server ，照上述規則就只有 stun server 能和你溝通，那何來的點對點溝通呢？
這時候要採用 webrtc 大不如就直接用 websocket

# stun (session reversal utilities for nat)

功用就是，把我的 public ip 和 port 回傳給我，基本上就是根本沒做什麼事，所以 google 免費提供此服務

# turn (traversal using relays around nat)

stun 進化版，當你的 ip 被 nat 的 symmetric nat 機制擋下來的時候，你就需要一個像代理伺服器的東西，什麼東西都經過他，也沒做什，就是把封包轉送給你，但是這就很貴

# ice (inteactive Connectivity Establishment)

蒐集 ice candidates 可能的端點，有可是 public ip 有可能是 private ip
有沒有可能兩個溝通的端點，在同一個內網，這樣舊部需要透過外部 ip 來溝通，也就不須透過 nat，我把這些 ice candidates 蒐集起來所以要走哪一個 ip 都由你選擇，可能一個會通一個不會通
