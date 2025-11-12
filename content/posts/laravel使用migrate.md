---
title: laravel使用migrate
date: 2020-03-12 16:15:40
tags:
- artisan
- migrate
categories:
- [後端路線,framework,laravel]
---
# 前言
假設你的資料表的建好了，你想要對資料庫做修改，但是資料不能清空，所以不能修改現有的檔案再migrate一次，你只能新增一個migration來修改資料庫的表格
# 使用情境

## 修改資料表的欄位名稱

    https://stackoverflow.com/questions/51130611/rename-column-in-laravel-using-migration
    