---
title: laravel使用Socialite
date: 2020-01-24 21:07:04
tags:
  - Socialite
  - oauth
categories:
  - [後端語言, framework, laravel]
---

# 前言

如果不提供第三方登入，很少使用者會愿意在新網站填入帳號密碼，會有安全願慮。Laravel Socialite 提供了簡單的 OAuth 登入整合，支援 GitHub、Google、Facebook 等常見平台。

# 安裝

```bash
composer require laravel/socialite
```

# 設定（以 GitHub 為例）

## 建立 GitHub OAuth App

到 GitHub 設定頁面 **Settings > Developer settings > OAuth Apps > New OAuth App**，填入：

- **Homepage URL**：`http://localhost`
- **Authorization callback URL**：`http://localhost/auth/github/callback`

建立完成後取得 `Client ID` 和 `Client Secret`。

![github申請 oAuth 畫面](/images/github申請oAuth畫面.PNG)

## 配置 .env

```
GITHUB_CLIENT_ID=your-client-id
GITHUB_CLIENT_SECRET=your-client-secret
GITHUB_REDIRECT_URI=http://localhost/auth/github/callback
```

## 配置 config/services.php

```php
'github' => [
    'client_id'     => env('GITHUB_CLIENT_ID'),
    'client_secret' => env('GITHUB_CLIENT_SECRET'),
    'redirect'      => env('GITHUB_REDIRECT_URI'),
],
```

# 路由設定

```php
Route::get('/auth/github', [AuthController::class, 'redirectToGithub']);
Route::get('/auth/github/callback', [AuthController::class, 'handleGithubCallback']);
```

# Controller 實作

```php
use Laravel\Socialite\Facades\Socialite;

// 將使用者導向 GitHub 登入
public function redirectToGithub()
{
    return Socialite::driver('github')->redirect();
}

// GitHub 回調，處理登入邏輯
public function handleGithubCallback()
{
    $githubUser = Socialite::driver('github')->user();

    $user = User::updateOrCreate(
        ['email' => $githubUser->email],
        [
            'name'            => $githubUser->name,
            'github_id'       => $githubUser->id,
            'profile_photo'   => $githubUser->avatar,
        ]
    );

    Auth::login($user, true);

    return redirect('/dashboard');
}
```

# 注意事項

## localhost 與 127.0.0.1 不可混用

GitHub OAuth callback URL 必須與瀏覽器實際存取的 URL **完全一致**。如果 GitHub 設定的 callback 是 `http://localhost/...`，就必須用 `http://localhost` 進入網站，不能用 `http://127.0.0.1`，否則會出現 `InvalidStateException`。

參考：[Laravel Socialite: InvalidStateException](https://stackoverflow.com/questions/30660847/laravel-socialite-invalidstateexception)
