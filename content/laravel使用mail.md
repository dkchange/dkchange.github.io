---
title: laravel使用mail
date: 2019-12-16 21:20:09
tags:
  - mail
categories:
  - [後端路線, framework, laravel]
---

# 前言

Laravel 內建了簡潔的郵件 API，支援多種驅動（SMTP、Mailgun、Mailtrap 等）。本篇介紹如何在開發環境使用 Mailtrap 測試寄信，以及如何建立 Mailable 類別傳遞資料給 Markdown 模板。

# 開發測試

Laravel 推薦使用 [Mailtrap](https://mailtrap.io/inboxes) 作為開發測試郵箱。
當你的郵件訊息寄到一個「假的」郵箱，而你可以在一個真的郵件客戶端檢視它們。

找到設置和下拉菜單「Demo inbox > SMTP Settings > Integrations > Laravel」
將資料填入 `.env`：

```
MAIL_MAILER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=no-reply@example.com
MAIL_FROM_NAME="${APP_NAME}"
```

# 指令

下指令會在 `resources/views/email/contact/` 底下產生 Markdown 模板資料夾與檔案：

```bash
php artisan make:mail ContactFormMail --markdown=email.contact.contact-form
```

建完之後在要用的 controller 裡面呼叫：

```php
Mail::to('test@test.com')->send(new ContactFormMail($data));
```

在 `ContactFormMail` 裡面修改，這樣資料才能傳進 view：

```php
public $data;
public function __construct($data)
{
    $this->data = $data;
}
```

在 Markdown view（`resources/views/email/contact/contact-form.blade.php`）中可透過 `$data` 存取傳入的資料。