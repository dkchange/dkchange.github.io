---
title: 使用webRTC
date: 2020-11-13 12:41:05
tags:
categories:
---

# 參考資料

[WebRTC Crash Course（YouTube）](https://youtu.be/FExZvpVvYxA)
[30 天之即時網路影音開發攻略（iThelp）](https://ithelp.ithome.com.tw/users/20089358/ironman/1923)（連結已失效）

# 前言

[[WebRTC]]（Web Realtime Communication）可以讓兩個瀏覽器之間 P2P 直接溝通，而中間不需要再透過 server，達到的目的就是快速。

# SDP（Session Description Protocol）

在兩個瀏覽器溝通之前，必須先傳給對方用 [[WebRTC]] 溝通所需資料，這個過程我們稱作 Signaling，而這些資料我們稱為 [[SDP]]。

Signaling 這個動作可以藉由任何管道，可以藉由通訊軟體、[[WebSocket]]，只要有辦法傳送給對方即可，那通常大家都會採用 WebSocket 來建一個 Signaling server。

SDP 裡面會包含很多的資料，傳輸的格式、加密的方法、ICE candidates 好多好多資料。

A → B A 把 Offer（SDP）給 B
B ← A B 收到之後，用這個 SDP 再創建一個 SDP Answer 給 A

# NAT（Network Address Translation）

如果你有 public IP 一切都變得簡單，但大部分的人都沒有，那我們就需要透過 NAT，NAT 簡單來說就是一張表，當你要和外部溝通時，他會建立對應的表，那收到回應時，我們就會來查這張表
你的 IP | 你的 Port | Router IP | Router Port | Host IP | Host Port
那不同的 Router 有不同的機制

- Full Cone NAT：不論你之前有沒有訪問過這 IP 都允許他和你溝通
- Address Restricted NAT：只要你之前有訪問過這 IP，就允許他和你溝通
- Port Restricted NAT：只要你之前有訪問過這 IP 而且 Port 是對的，就允許他和你溝通
- Symmetric NAT：比必須要 Router 出去的 IP 和 Port，訪問的主機 IP 和 Port 這四個東西都相同，我才允許你回應

然而 WebRTC 跟 Symmetric NAT 相較之下比較沒那麼相容，因為 WebRTC 需要有一個 STUN server，照上述規則就只有 STUN server 能和你溝通，那何來的點對點溝通呢？
這時候要採用 WebRTC 大不如就直接用 WebSocket

# STUN（Session Traversal Utilities for NAT）

功用就是把我的 public IP 和 Port 回傳給我，基本上就是沒做什麼事，所以 Google 免費提供此服務。

# TURN（Traversal Using Relays around NAT）

STUN 進化版，當你的 IP 被 NAT 的 Symmetric NAT 機制擋下來的時候，你就需要一個像代理伺服器的東西，什麼東西都經過他，也沒做什麼，就是把封包轉送給你，但是這就很貴。

# ICE（Interactive Connectivity Establishment）

蒐集 ICE candidates 可能的端點，有可能是 public IP 有可能是 private IP
有沒有可能兩個溝通的端點，在同一個內網，這樣就不需要透過外部 IP 來溝通，也就不須透過 NAT，我把這些 ICE candidates 蒐集起來所以要走哪一個 IP 都由你選擇，可能一個會通一個不會通
