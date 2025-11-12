---
title: laravel使用mail
date: 2019-12-16 21:20:09
tags:
- mail
categories:
- [後端路線,framework,laravel]
---

# 前言

# 開發測試
laravel預設的網站https://mailtrap.io/inboxes#
將你的郵件訊息寄到一個「假的」郵箱，而你可以在一個真的郵件客戶端檢視它們。
找到頁籤和下拉選單`Demo inbox/smtp_settings/Integrations/laravel`
將資料填進.env

    MAIL_USERNAME=
    MAIL_PASSWORD=
    MAIL_ENCRYPTION=



# 指令
下指令會在view底下建資料夾

    php artisan make:mail ContactFormMail --markdown=email.contact.contact-form

建完之後在要用的controller裡面

```php
Mail::to('test@test.com')->send(new ContactFormMail($data));
```
在ContactFormMail裡面修改，這樣資料才傳的進view

```php
    public $data;
    public function __construct($data)
    {
        $this->data = $data;
    }
```