---
title: laravel使用pusher
date: 2020-03-12 16:22:27
tags:
  - pusher
  - websocket
categories:
  - 後端語言
  - framework
  - laravel
---

# 前言

工作中有時候需要即時通訊的機能，因為公司不可能每個服務都自己開發，所以代替方案是使用 Pusher 這個第三方 WebSocket 服務來實現即時推送。前端部分使用 Vue 配合 Laravel Echo 來監聽事件。

參考教學：https://pusher.com/tutorials/chat-laravel

# 安裝套件

```bash
composer require pusher/pusher-php-server
npm install --save laravel-echo pusher-js
```

# 設定

## 註冊 Pusher 帳號

到 [pusher.com](https://pusher.com) 註冊帳號，建立一個 Channels App，取得以下四組認證資訊：

- `PUSHER_APP_ID`
- `PUSHER_APP_KEY`
- `PUSHER_APP_SECRET`
- `PUSHER_APP_CLUSTER`

## 配置 .env

```
BROADCAST_DRIVER=pusher

PUSHER_APP_ID=your-app-id
PUSHER_APP_KEY=your-app-key
PUSHER_APP_SECRET=your-app-secret
PUSHER_APP_CLUSTER=ap3
```

## 開啟 Broadcast Service Provider

在 `config/app.php` 中，取消以下這行的註解：

```php
App\Providers\BroadcastServiceProvider::class,
```

## 配置 broadcasting.php

`config/broadcasting.php` 中 Pusher 的設定會自動讀取 `.env`，不需要手動修改。

# 建立廣播事件

## 建立 Event

```bash
php artisan make:event MessageSent
```

在 `app/Events/MessageSent.php` 中，實作 `ShouldBroadcast` 介面：

```php
<?php

namespace App\Events;

use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;

class MessageSent implements ShouldBroadcast
{
    use SerializesModels;

    public $message;

    public function __construct($message)
    {
        $this->message = $message;
    }

    public function broadcastOn()
    {
        return new \Illuminate\Broadcasting\Channel('chat');
    }
}
```

## 觸發事件

在 controller 中觸發廣播事件：

```php
broadcast(new MessageSent($message));
// 或使用
event(new MessageSent($message));
```

# 前端監聽（Vue + Laravel Echo）

## 配置 bootstrap.js

在 `resources/js/bootstrap.js` 中加入：

```js
import Echo from "laravel-echo"
window.Pusher = require("pusher-js")

window.Echo = new Echo({
  broadcaster: "pusher",
  key: process.env.MIX_PUSHER_APP_KEY,
  cluster: process.env.MIX_PUSHER_APP_CLUSTER,
  encrypted: true,
})
```

## 在 Vue 組件監聽事件

```js
mounted() {
    Echo.channel('chat')
        .listen('MessageSent', (e) => {
            console.log(e.message);
            this.messages.push(e.message);
        });
}
```

## 編譯前端資源

```bash
npm run dev
```
