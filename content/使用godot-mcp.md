---
title: 使用godot Mcp
date: 2025-11-17T16:55:51+08:00
draft: true
---

# 前言

因為vibe coding 正流行，總要讓ai agent 有辦法跟godot編輯器互動，所以上網找了幾個mcp工具

- https://github.com/Coding-Solo/godot-mcp
- https://github.com/ee0pdt/godot-mcp
- https://gdaimcp.com/

最後選用的Coding-Solo/godot-mcp 搭配claude code

# 使用方式

使用claude code 內建的指令 它自己會建立~/Stanley/.claude.json，之後你就能用到這個MCP工具了

## claude code 添加使用者級別的MCP

```sh
claude mcp add --scope user --transport stdio godotmcp -- node /home/stanley/godot-mcp/build/index.js
```

## claude code 添加特定專案級別的MCP

在指定的專案資料夾底下

```sh
claude mcp add --transport stdio godotmcp -- node /home/stanley/godot-mcp/build/index.js
```
