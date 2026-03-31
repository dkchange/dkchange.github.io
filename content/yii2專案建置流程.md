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

1. 安裝 composer

   目前這個時間點已經有 composer2 了

2. 安裝 yii2

   ```console
   //XXX是專案名
   composer create-project --prefer-dist yiisoft/yii2-app-advenced XXX
   ```

3. 挑選後台樣板

   下載 sb admin

4. 設定網址

   - 後台http://backend.localhost:8888/
   - 前台http://frontend.localhost:8888/

5. 初始化專案

   php init

6. 註冊使用者

   到前台http://frontend.localhost:8888/註冊會發現資料庫不存在

7. 創建資料庫

   填寫資料庫名 ， 選擇編碼 utf8mb4_general_ci

8. 跑 migrate

   跑環系統會新增一個 user 的 table，成功之後便可以正常創建帳號

9. 驗證帳號

   yii2 的 mail 設定檔裡面會將驗證的 mail 寫進 frontend/runtime/mail
   裡面的字串是屬於特殊格式的，他每換一行會多加個=,每次遇到=會多加一個 3D
   所以去掉多餘的= 3D 之後，將網址貼上 browser 就可以驗證成功

# 套用樣板

將資源整合進 AppAsset.php

1. 共用的資源和 header footer 整合進 layout/main.php
2. yii2 預設整合 bootstrap3 如果要關掉預設，要在設定檔

```php
    'assetManager'=>[
        "bundles"=>[
            \yii\bootstrap\BootstrapAsset::class=>false
        ]
    ]
```

```console
    composer remove yiisoft/yii2-bootstrapt
```

3. yii2 也內建 jquery，如果需要的話

```php
    //AppAsset.php
    public $depends=[
        JqueryAsset::class
    ]
```

4. yii2 也提供 bootstrap4 套件

```console
    composer require yiisoft/yii2-bootstrapt4
```

5. 由於 sbAdmin 的 sb-admin.css 已經把 boostrap4 包進去了，所以我們不需要額外引進 bootstrap4 的 css，我們只需要 bootstrap4 的 js 即可

```php
    //AppAsset.php
    public $depends=[
        BootstrapPluginAsset::class //just js
    ]
```

6. 又因為 YiiAsset 包含 bootstrap.css，但我們只需要 js 即可

```php
    'assetManager'=>[
        "bundles"=>[
            \yii\bootstrap\BootstrapAsset4::class=>false
        ]
    ]
```

7. 刪掉多餘的檔案
8. 開啟 pretty url

# 設計 db(這步驟要仔細想，花時間)

1. 利用 yii2 的 migratte 工具建立資料表， 如果在使用 docker 要注意權限問題，我新增一個一樣的使用者 stanley 在容器裡面，再切換使用者，讓我可以對 migration 檔案做修改和執行
2. 用 gii 創建 model 建 query
3. 用 gii 創建 crud controller search view
4. 整合 view 頁面

# 後台表單(關係到 ui 更花時間)

1. 安裝 wysiwyg

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
4. 因為採用 bootstrap4 所以需要客製 ActionColumn

# rest api

在 yii2 新增 restapi 的方法大致上有兩種，一種是你採用 yii2-base 的話，你會想要建 api module 來擴展此功能，那如果你是用 yii2-advenced 的話，建議是複製一個新的資料夾，像 backend frontend 一樣可以有自己的設定檔，同時 module 可以當作留存各版本 api 的擴充
https://www.youtube.com/watch?v=TPOyQ_W8VdA
https://stackoverflow.com/questions/54009254/yii2-rest-api-location

1. 新增欄位
   https://segmentfault.com/a/1190000017507487
   https://www.programmersought.com/article/7422457902/
   https://stackoverflow.com/questions/28238595/yii2-authkey-whats-the-purpose

2. 把前端 frontend 資料夾用來作 api

3. 建立 resources 格式化輸出格式

4. //todo CROS 設定的時候卡住了

# authentication

yii 內建了幾種認證方式

1. 基於 session
2. 基於 accessToken(適用於 stateless)
3. 基於 cookie enableAutoLogin 设置为 true，就会使用 cookie 登录，配合 cookieValidationKey 防止 cookie 被串改，

https://ithelp.ithome.com.tw/articles/10197166

# authorizaion

https://www.yiichina.com/doc/guide/2.0/security-authorization
@ is the special symbol that was recognized by Yii to identify authenticated users.

? is used to identify unauthorized users
other than @ and ? are considered to be the name of the currently logged in users. so here admin is the name of the user who will holds the permission to do admin and delete actions in your controller.

# Role-based Access Control

https://www.youtube.com/watch?v=vLb8YATO-HU
https://www.youtube.com/watch?v=tMNJi9jaCrY

# workerman

目前專案想要達成的部份有兩個

1. 普通聊天
2. webrtc

那 workerman 有提供這兩個範例檔
所以就將其做個整合
https://github.com/nick-bai/laychat

ws = new WebSocket("wss://localhost:4443/socket");
// 当 socket 连接打开时，输入用户名
ws.onopen = function(data){
console.log(data)
};
// 当有消息时根据消息类型显示不同信息
ws.onmessage = function(data){
console.log(data)
};
ws.onclose = function() {
console.log("连接关闭，定时重连");
connect();
};
ws.onerror = function() {
console.log("出现错误");
};
