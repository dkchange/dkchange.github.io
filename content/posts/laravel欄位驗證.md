---
title: laravel欄位驗證
date: 2019-11-08 02:15:08
tags: 欄位驗證
categories:
- [後端路線,framework,laravel]
---
# 前言
欄位驗證一般都放在request裡或controller裡，我偏好放在request裡面就做驗證，
為什麼欄位驗證要和request綁一起呢?
因為現在幾乎前後端分離，假如有天同時要做傳統的頁面和分離API頁面，兩個不同的controller才會請求都有被驗證，而且不用進到controller就擋下來，省時

# 特別規則

*sometimes
有填的時候再做驗證
