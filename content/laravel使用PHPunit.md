---
title: laravel使用PHPunit
date: 2019-12-19 21:44:48
tags:
  - PHPunit
categories:
  - 後端語言
  - framework
  - laravel
---

# 前言

TDD測試驅動開發，因為測試是下己寫的，所以不會預期到特殊狀況，例如你使用者輸入什麼都會發生的錯誤，或是某個時間点会發生的錯誤，經驗不够就是程式碼越寫越大之後，來不及修正。所以就來寫測試吧

# 設定

如果你有裝 telescope外掛件，在 `phpunit.xml` 作修改，那這個嗎時就是 PHPunit 執行的環境，在上面的修改要加上廣設置.env，然後我們每次執行測試都是全新的環境，所以也要改變 DB 的設定等

```xml
<php>
    <server name="TELESCOPE_ENABLED" value="false">
    <server name="DB_CONNECTION" value="sqlite">
    <server name="DB_DATABASE" value=":memory:">
</php>
```

# 執行

執行 vendor 底下的執行檔，經過代表成功

```bash
vendor/bin/phpunit
vendor/bin/phpunit --filter only_logged_in_user_can_edit_profile
vendor/bin/phpunit --filter UsersTest
vendor/bin/phpunit --unit UsersTest
```

清除畫面

```bash
clear && !!
```

因為為次都需要打這樣的指令會很降低效率，所以將這兩個指令合併為一，取一個別名

```bash
alias pu='clear && vendor/bin/phpunit'
alias pf='clear && vendor/bin/phpunit --filter'
```

設定完之後就會在 `~/.zshrc` 裡到我們的設定。
如果 windows 下不可能就是指就方法的不一樣
每次進入這樣的 CMD 都需要再次輸入一次上述指令
我們可以新增一個 `.bash_profile` 檔案，然後放等

```bash
alias pu='clear && vendor/bin/phpunit'
alias pf='clear && vendor/bin/phpunit --filter'
```

將它傓存後，我們首先

```bash
source ~/.bash_profile
```

就可以使用了

# 撰寫

```bash
php artisan make:test BirthdayTest
```

`tests\Feature` 資料夾下建立我們加能檔案，我們通常會建立一個 controller 作為一們，一個 controller 建立一個功能，而且是同步進行，
**程式碼測試先行開發首先**了，因為在程式式的當下你才會考慮到可能狀況，之後有工廠就很難全部考慮到，需要加上工廠factory就可以拿來用了，每個 function 名稱都需要一目瞭然，名字可以很長也沒關係， e.g.`only_logged_in_user_can_edit_profile`

`tests\Unit` 資料夾下建立我們加能檔案，我們通常會建立一個 model 作為一偉， unit則feature的區別是， feature包含了到使用UI互動的測試，如你能看到網頁內容測試到這裡的連結

## 常用的方法

```php
// 這一行會幫我們 migrate, 且是清空
use RefreshDatabase;
// 所有 function 造行時都在設置先達到的功能
private function setUp():void{
    parent::setUp();
    Event::fake();
}

//only_logged_in_user_can_edit_profile
$response = $this->get('/edit')->assertRedirect('login');

//only_authenticate_users_can_edit_profile
$this->actingAs(factory(User::class)->create());
$response = $this->get('/edit')->assertOk();

//an_user_added_through_the_form
//laravel 內幫 http 請求錯誤處理機制， 有真正的錯誤
$this->withoutExceptionHandling();
//假設 event
Event::fake();
$this->actingAs(factory(User::class)->create([
    'email' => 'admin@admin.com'
]));
$this->post('/add',[
    'name' => 'test user',
    'email' => 'test@test.com'
]);
$this->assertCount(1,User::all());

//an_email_is_required
$response = $this->post('add',array_merge($this->data(),[
    'email' => ''
]));
```

---

# 使用情境

## 圖片測試

Q: How to create a phpUnit test for images?
A: I’d prefer not to test such things，圖片上傳測試建議不要寫，可以用 `Event::fake()` 去 mock 掉上傳事件。

參考：https://stackoverflow.com/questions/39041876/how-to-create-a-phpunit-test-for-images
