---
title: yii2專案建置流程
date: 2021-03-05 10:48:22
tags:
categories:
---

# 前言

本篇將紀錄專案從 0 開始的過程
[參考來源](https://www.youtube.com/watch?v=eQdDBhQpU9o)

# 前置動作

1. 安裝 [[Composer]]

   目前這個時間點已經有 Composer 2 了

2. 安裝 [[Yii2]]

   ```console
   //XXX是專案名
   composer create-project --prefer-dist yiisoft/yii2-app-advanced XXX
   ```

3. 挑選後台樣板

   下載 SB Admin

4. 設定網址
   - 後台 http://backend.localhost:8888/
   - 前台 http://frontend.localhost:8888/

5. 初始化專案

   php init

6. 註冊使用者

   到前台 http://frontend.localhost:8888/ 註冊會發現資料庫不存在

7. 創建資料庫

   填寫資料庫名，選擇編碼 utf8mb4_general_ci

8. 跑 Migrate

   跑Migration系統會新增一個 User 的 Table，成功之後便可以正常創建帳號

9. 驗證帳號

   Yii2 的 Mail 設定檔裡面會將驗證的 Mail 寫進 frontend/runtime/mail
   裡面的字串是屬於特殊格式的，他每換一行會多加個 =，每次遇到 = 會多加一個 3D
   所以去掉多餘的 = 3D 之後，將網址貼上 Browser 就可以驗證成功

# 套用樣板

將資源整合進 AppAsset.php

1. 共用的資源和 Header Footer 整合進 layout/main.php
2. [[Yii2]] 預設整合 Bootstrap3，如果要關掉預設，要在設定檔

```php
    'assetManager'=>[
        "bundles"=>[
            \yii\bootstrap\BootstrapAsset::class=>false
        ]
    ]
```

```console
    composer remove yiisoft/yii2-bootstrap
```

3. [[Yii2]] 也內建 jQuery，如果需要的話

```php
    //AppAsset.php
    public $depends=[
        JqueryAsset::class
    ]
```

4. [[Yii2]] 也提供 Bootstrap4 套件

```console
    composer require yiisoft/yii2-bootstrap4
```

5. 由於 SB Admin 的 sb-admin.css 已經把 Bootstrap4 包進去了，所以我們不需要額外引進 Bootstrap4 的 CSS，我們只需要 Bootstrap4 的 JS 即可

```php
    //AppAsset.php
    public $depends=[
        BootstrapPluginAsset::class // just JS
    ]
```

6. 又因為 YiiAsset 包含 Bootstrap.css，但我們只需要 JS 即可

```php
    'assetManager'=>[
        "bundles"=>[
            \yii\bootstrap\BootstrapAsset4::class=>false
        ]
    ]
```

7. 刪掉多餘的檔案
8. 開啟 Pretty URL

# 設計 DB（這步驟要仔細想，花時間）

1. 利用 [[Yii2]] 的 Migration 工具建立資料表，如果在使用 [[Docker]] 要注意權限問題，我新增一個一樣的使用者 Stanley 在容器裡面，再切換使用者，讓我可以對 Migration 檔案做修改和執行
2. 用 [[Gii]] 創建 Model 建 Query
3. 用 [[Gii]] 創建 CRUD Controller Search View
4. 整合 View 頁面

# 後台表單（關係到 UI 更花時間）

1. 安裝 WYSIWYG

```console
composer require 2amigos/yii2-ckeditor-widget:2.1.0
```

```php
//form.php
    <?= $form->field($model, 'description')->widget(CKEditor::class,[
            'options' => ['row'=>6],
            "preset"=>"basic"
    ]) ?>
```

2. create_at create_by 欄位

```php
    public function behaviors()
    {
        return [
            TimestampBehavior::class,
            BlameableBehavior::class
        ];
    }
```

3. 表單驗證
4. 因為採用 Bootstrap4 所以需要客製 ActionColumn

# REST API

在 [[Yii2]] 新增 REST API 的方法大致上有兩種，一種是你採用 Yii2-basic 的話，你會想要建 API Module 來擴展此功能，那如果你是用 Yii2-advanced 的話，建議是複製一個新的資料夾，像 Backend、Frontend 一樣可以有自己的設定檔，同時 Module 可以當作留存各版本 API 的擴充
[YouTube 教學](https://www.youtube.com/watch?v=TPOyQ_W8VdA)
[Stack Overflow 討論](https://stackoverflow.com/questions/54009254/yii2-rest-api-location)

1. 新增欄位
   [SegmentFault 教學](https://segmentfault.com/a/1190000017507487)
   [Programmersought 教學](https://www.programmersought.com/article/7422457902/)
   [Stack Overflow 討論](https://stackoverflow.com/questions/28238595/yii2-authkey-whats-the-purpose)

2. 把前端 Frontend 資料夾用來作 API

3. 建立 Resources 格式化輸出格式

4. // TODO CORS 設定的時候卡住了

# Authentication

[[Yii2]] 內建了幾種認證方式

1. 基於 Session
2. 基於 AccessToken（適用於 Stateless）
3. 基於 Cookie，enableAutoLogin 設置為 true，就會使用 Cookie 登入，配合 cookieValidationKey 防止 Cookie 被篡改

[iThelp 教學](https://ithelp.ithome.com.tw/articles/10197166)

# Authorization

[Yii2 中文文檔](https://www.yiichina.com/doc/guide/2.0/security-authorization)
@ is the special symbol that was recognized by [[Yii2]] to identify authenticated users.

? is used to identify unauthorized users
other than @ and ? are considered to be the name of the currently logged in users. So here admin is the name of the user who will hold the permission to do admin and delete actions in your controller.

# Role-based Access Control

[YouTube 教學](https://www.youtube.com/watch?v=vLb8YATO-HU)
[YouTube 教學](https://www.youtube.com/watch?v=tMNJi9jaCrY)

# Workerman

目前專案想要達成的部分有兩個

1. 普通聊天
2. [[WebRTC]]

那 Workerman 有提供這兩個範例檔
所以就將其做個整合
[GitHub 專案](https://github.com/nick-bai/laychat)

```javascript
ws = new WebSocket("wss://localhost:4443/socket")
// 當 socket 連線開啟時，輸入使用者名稱
ws.onopen = function (data) {
  console.log(data)
}
// 當有訊息時根據訊息類型顯示不同資訊
ws.onmessage = function (data) {
  console.log(data)
}
ws.onclose = function () {
  console.log("連線關閉，定時重連")
  connect()
}
ws.onerror = function () {
  console.log("出現錯誤")
}
```
