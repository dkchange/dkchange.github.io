---
title: 試用claude code
date: 2025-11-12 11:27:18
tags:
categories:
---

# 前言
好久沒更新blog了，都快忘記怎麼更新，在此做個紀錄

# 免費的api
網路上會有一些三方的平台提供一些免費的額度給你試用，在 **~/.claude/settings.json**設定

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "三方提供的網址",
    "ANTHROPIC_AUTH_TOKEN": "你的API_KEY"
  }
}
```

設定完之後，claude code 自己會偵測到檔案，直接使用可