---
title: 從hugo到quartz
date: 2026-03-31
tags:
  - Github
categories:
---

# 前言
之前一直想嘗試obsidian(黑曜石)這個筆記軟體，想說既然都換筆記軟體了，GitHub page是不是也也可以跟著換，查了一下就決定選用quartz

# 創建筆記儲存庫
在quartz資料夾底下

	npx quartz create

# 建置且啟動本地服務

	npx quartz build --serve

# 主題

# 推上GitHub
因為我們是clone quartz專案來改，所以origin 要改成自己的倉庫
quartz專案會更新，所以upstream不動，pull時才會拿到quartz的最新版

	git remote set-url origin git@github.com:dkchange/dkchange.github.io.git