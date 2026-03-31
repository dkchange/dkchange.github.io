---
title: laravel欄位驗證
date: 2019-11-08 02:15:08
tags: 欄位驗證
categories:
  - [後端路徑,framework,laravel]
---

# 前言

欄位驗證一般放在 Request 或 controller 裡，建議放在 Request 裡面做驗證。

為什麼要把驗證放在 Request 而不放在 Controller？

因為 Laravel 的 Form Request 會在進入 Controller 之前就執行驗證，不合規的請求會直接被擋下來（返回錯誤訊息），不會進到 Controller，讓 Controller 只需專注在商業邏輯。

此外，現在幾乎都做前後端分離，不同的 Controller（網頁前端和 API）都需要相同的驗證邏輯，把驗證抽到 Request 也可以重用。

## 建立 Form Request

```bash
php artisan make:request StoreUserRequest
```

產生的檔案位於 `app/Http/Requests/StoreUserRequest.php`，裡面有兩個方法：

```php
public function authorize(): bool
{
    return true; // 改成 true，或加入授權邏輯
}

public function rules(): array
{
    return [
        'name'  => 'required|string|max:255',
        'email' => 'required|email|unique:users,email',
        'age'   => 'nullable|integer|min:0',
    ];
}
```

Controller 裡只需型別提示帶入 Request 即可，驗證自動執行：

```php
public function store(StoreUserRequest $request)
{
    // 能進到這裡代表驗證已通過
    User::create($request->validated());
}
```

驗證失敗時：
- **網頁請求**：自動重導回上一頁，並帶上錯誤訊息（可用 `$errors` 在 blade 取得）
- **API 請求**：自動回傳 `422 Unprocessable Entity` 加上 JSON 錯誤訊息

## 自訂錯誤訊息

在 Form Request 裡加上 `messages()` 方法：

```php
public function messages(): array
{
    return [
        'name.required'  => '姓名為必填欄位',
        'email.required' => '電子郵件為必填欄位',
        'email.unique'   => '此電子郵件已被使用',
    ];
}
```

# 特別提醒

## `sometimes`

有填的時候才做驗證，欄位不存在於請求中時直接跳過：

```php
'phone' => 'sometimes|string|max:20',
```

## `nullable`

允許欄位為 null，與 `sometimes` 的差異：
- `sometimes`：欄位完全不在請求裡時跳過驗證
- `nullable`：欄位存在但值為空（null 或空字串）時允許通過

```php
'bio' => 'nullable|string|max:500',
```

## `required_if` / `required_with`

條件式必填：

```php
// 當 role 為 admin 時，department 必填
'department' => 'required_if:role,admin',

// 當 password 有值時，password_confirmation 必填
'password_confirmation' => 'required_with:password',
```