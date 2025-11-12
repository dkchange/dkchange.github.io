---
title: laravel使用eloquent
date: 2020-03-06 09:59:50
tags:
categories:
---
# 前言

eloquent是遵循 active record pattern  ，但你去要資料，違反了 tell dont ask 原則
doctrine是遵循 data mapper pattern


# 表格的default value
在migration就設定default value
- ->default($value)	Specify a "default" value for the column
或者
```php
    const ROLE_PUBLISHER = 1;

    protected $attributes = [
        'role_id' => self::ROLE_PUBLISHER,
    ];
```

https://stackoverflow.com/questions/39912372/how-to-set-the-default-value-of-an-attribute-on-a-laravel-model

# many to many relationship
https://www.youtube.com/watch?v=akKWC_vP7sE

# fake relationship

https://www.youtube.com/watch?v=rx1DQBE01b0

# Laravel数据库操作的三种方式

https://www.youtube.com/watch?v=3TJfR1Ta4GU

# laravel 配置MySQL讀寫分離

https://zhuanlan.zhihu.com/p/123092691


# model如何知道table的欄位

https://stackoverflow.com/questions/23396138/laravel-eloquent-model-attributes

