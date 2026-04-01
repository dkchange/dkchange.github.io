---
title: laravel資訊安全
date: 2021-12-30 10:24:05
tags:
categories:
  - 後端
---

# 前言

[YouTube 教學 - How your Laravel application can get hacked](https://www.youtube.com/watch?v=3k8n6Vi4rg4)

# SQL Injection

## 攻擊說明

如果你用 DB::raw 但沒有適當處理參數就直接拼接字串，將可能導致 SQL Injection。

```php
// 危險寫法 - 容易被 SQL Injection
$userId = $_GET['id'];
$user = DB::select(DB::raw("SELECT * FROM users WHERE id = $userId"));
```

使用 sqlmap 測試 URL 是否有 SQL Injection：

```bash
sqlmap -u "https://example.com/api/users?id=1" --batch
```

## 防禦方式

### 1. 使用 Eloquent 或 Query Builder 的參數綁定

Laravel 的 Eloquent 和 Query Builder 預設會使用 Parameter Binding，自動防禦 SQL Injection：

```php
// ✅ 安全寫法 - 自動參數綁定
$user = DB::table('users')->where('id', $userId)->first();
$user = User::where('email', $email)->first();
```

### 2. DB::raw 的正確用法

若必須使用 `DB::raw`，務必搭配參數綁定：

```php
// ✅ 安全寫法 - 使用 binding
$users = DB::select(
    DB::raw("SELECT * FROM users WHERE id = :id"),
    ['id' => $userId]
);
```

### 3. 避免直接拼接使用者輸入

```php
// ❌ 危險
$query = "SELECT * FROM users WHERE name LIKE '%" . $name . "%'";

// ✅ 安全
$users = DB::table('users')->where('name', 'LIKE', "%{$name}%")->get();
```

# Object Injection

## 攻擊說明

當使用 `unserialize()` 反序列化不受信任的資料時，攻擊者可以構造惡意的序列化字串，觸發 `__destruct()` 或其他魔術方法執行任意程式碼。

### 範例1：phpggc 工具

[phpggc](https://github.com/ambionics/phpggc) 是一個 PHP gadget chain 產生工具，可以生成針對不同框架的 exploit。

### 範例2：危險的反序列化程式碼

```php
class Evil {
    public $cmd = 'id';

    public function __destruct() {
        shell_exec($this->cmd);
    }
}

// 攻擊者傳入惡意序列化字串
$data = $_POST['data'];
unserialize($data); // 觸發 __destruct，執行系統指令
```

### 範例3：phar 文件反序列化

phar 文件在某些 PHP 文件函數中會自動觸發反序列化，即使沒有直接呼叫 `unserialize()`：

```php
// 當 $filename 是 phar:// 協議時，會觸發反序列化
filesize($filename);
file_exists($filename);
```

攻擊流程：

1. 攻擊者上傳混合了序列化 payload 的圖片檔案
2. 伺服器對這個檔案呼叫 `filesize()` 或 `file_exists()`
3. 觸發反序列化，執行攻擊代碼

## 防禦方式

### 1. 不要反序列化來自使用者的資料

```php
// ❌ 危險 - 不要對使用者輸入進行反序列化
$data = unserialize($_POST['data']);

// ✅ 安全 - 使用 JSON
$data = json_decode($_POST['data'], true);
```

### 2. 使用 JSON 取代 PHP 序列化

JSON 不會觸發類別的魔術方法，相對安全：

```php
// 儲存資料時使用 JSON
$serialized = json_encode($data);

// 讀取資料時使用 JSON
$data = json_decode($serialized, true);
```

### 3. 限制上傳檔案類型

防止 phar 檔案被上傳：

```php
// 驗證上傳檔案
$request->validate([
    'file' => 'required|mimes:jpg,png,pdf|max:2048'
]);

// 禁止 phar 檔案
if (strpos($filename, 'phar://') === 0) {
    throw new Exception('Invalid file type');
}
```

### 4. 對檔案操作使用白名單路徑

```php
// ❌ 危險 - 使用者可控制檔案路徑
$path = $_GET['file'];
filesize($path);

// ✅ 安全 - 限制在特定目錄內
$safePath = storage_path('uploads/' . basename($_GET['file']));
if (file_exists($safePath)) {
    filesize($safePath);
}
```

# Laravel 漏洞案例

## CVE-2021-39165: Cachet SQL Injection

這是一個在 Cachet CMS（基於 Laravel 開發）中發現的 SQL Injection 漏洞，由資安研究員 P 牛發現。

### 漏洞成因

1. Cachet 的 `scopeSearch` 使用 `array_intersect` 做白名單檢查，但邏輯有漏洞
2. 只要輸入陣列中有任一個 key 在白名單裡，整個陣列就會被傳入 `where()`
3. 透過 Laravel 的 `addArrayOfWheres()` 機制，可以控制 `where()` 的第四個參數 `$boolean`
4. `$boolean` 參數沒有過濾，直接拼接進 SQL 語句，造成注入

### 漏洞 POC

```
GET /api/v1/components?name=1&1[0]=&1[1]=a&1[2]=&1[3]=or 'a'=? and 1=1) -- +
```

### 完整攻擊鏈

1. **前台 SQL Injection** → 取得管理員的 API Token
2. **後台 Twig SSTI** → 利用 `__env` 或 `app` 物件執行任意 PHP 代碼
3. **WAF 繞過** → 在關鍵字中插入控制字元 `%01` 繞過 WAF

### 詳細分析

建議閱讀原作者的完整分析文章：

[[CVE-2021-39165: 從一個Laravel SQL注入漏洞開始的Bug Bounty之旅]](https://www.leavesongs.com/PENETRATION/cachet-from-laravel-sqli-to-bug-bounty.html)

### 防禦建議

1. 對自訂的 scope 方法進行嚴格的參數驗證
2. 使用白名單時，確保邏輯正確（不能只檢查「部分匹配」）
3. 避免將使用者輸入直接傳入 Model 的 where 條件
4. 定期更新 Laravel 及第三方套件

---

# 資訊安全最佳實踐

1. **永遠不信任使用者輸入** - 所有外部資料都需要驗證和過濾
2. **使用框架提供的安全機制** - Eloquent、Query Builder 的參數綁定
3. **避免危險函數** - `unserialize()`、`eval()`、`shell_exec()` 等
4. **定期更新依賴** - `composer update` 並關注安全公告
5. **程式碼審查** - 重點檢查資料庫查詢、檔案操作、模板渲染
6. **使用安全測試工具** - sqlmap、phpggc、安全掃描器
