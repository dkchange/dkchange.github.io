---
title: laravel使用migrate
date: 2020-03-12 16:15:40
tags:
  - artisan
  - migrate
categories:
  - 後端語言
  - framework
  - laravel
---

# 前言

Migration 是 Laravel 的資料庫版本控制工具，讓你用程式碼管理資料表的結構變更。每次需要修改資料庫結構時，應該建立一個新的 migration 檔案，而不是修改舊的，這樣可以保留完整的變更歷史，也方便團隊協作與環境同步。

# 基本指令

## 建立 migration 檔案

```bash
php artisan make:migration create_users_table
```

建立後，檔案會出現在 `database/migrations/` 目錄下。

## 執行 migration

```bash
php artisan migrate
```

## 回滾上一次的 migration

```bash
php artisan migrate:rollback
```

## 回滾所有 migration 並重新執行

```bash
php artisan migrate:fresh
```

> 注意：`migrate:fresh` 會刪除所有資料表再重建，正式環境請勿使用。

# 使用情境

## 修改資料表的欄位名稱

若資料表已有資料，不能直接修改舊的 migration，應新建一個 migration 來進行欄位更名。

首先安裝 doctrine/dbal（Laravel 9 以前需要）：

```bash
composer require doctrine/dbal
```

建立新的 migration：

```bash
php artisan make:migration rename_name_to_username_in_users_table
```

在 migration 檔案中使用 `renameColumn`：

```php
public function up()
{
    Schema::table('users', function (Blueprint $table) {
        $table->renameColumn('name', 'username');
    });
}

public function down()
{
    Schema::table('users', function (Blueprint $table) {
        $table->renameColumn('username', 'name');
    });
}
```

參考：https://stackoverflow.com/questions/51130611/rename-column-in-laravel-using-migration

## 修改欄位型別或屬性

使用 `change()` 方法修改欄位定義：

```bash
php artisan make:migration change_email_column_in_users_table
```

```php
public function up()
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('email', 100)->nullable()->change();
    });
}

public function down()
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('email', 255)->nullable(false)->change();
    });
}
```

## 新增欄位

```php
public function up()
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('phone')->nullable()->after('email');
    });
}

public function down()
{
    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('phone');
    });
}
```
