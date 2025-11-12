---
title: laravel使用PHPunit
date: 2019-12-19 21:44:40
tags:
- PHPunit
categories:
- [後端路線,framework,laravel]
---
# 前言
TDD測試驅動開發，因為測試是自己寫的，所以不會預料到特殊情況，例如你使用者輸入什麼會發生的錯,或是某個時間點會發生的錯，能做的是就是確保程式寫大之後，東改西改***某個流程***不會壞掉，所以就來寫測試吧
# 設定
如果你有裝Telescope會報錯，在`phpunit.xml`作修改，那這個檔案時就是PHPunit執行的環境，在上面的修改實際上會覆蓋.env，然後我們希望每次執行測時都是全新的環境，所以也要修改DB的設定檔

```xml
<php>
<server name="TELESCOPE_ENABLED" value="false">
<server name="DB_CONNECTION" value="sqlite">
<server name="DB_DATABASE" value=":memory:">
</php>
```

# 執行
執行vender底下的執行檔，綠燈代表成功

    vendor/bin/phpunit
    vendor/bin/phpunit --filter only_logged_in_user_can_edit_profile
    vendor/bin/phpunit --filter UsersTest
    vendor/bin/phpunit --unit UsersTest
    //清除畫面
    clear && !!
    那因為每次都需要打那長的指令會覺得麻煩，所以將這個指令取一個別名
    alias pu = 'clear && vendor/bin/phpunit'
    alias pf = 'clear && vendor/bin/phpunit --filter'
    設定完之後就會在~./zhsrc看到我們的設定
    那在windows底下可能就和我預期的不一樣
    每次重開終端機CMD都需要再重打一次上述指令
    我們可以新增一個.bash_profile檔案，裡面放著
    alias pu = 'clear && vendor/bin/phpunit'
    alias pf = 'clear && vendor/bin/phpunit --filter'
    那在要使用別名之前，我們就先
    source ~/.bash_profile
    就可以使用了


# 撰寫

    php artisan make:test BirthdayTest
`tests\Feature`資料夾底下建立我們功能檔案，我們通常會對一個controller作撰寫，一個controller對應一個功能，而且是同步進行，***編寫程式就要編寫測試***了，因為寫程式的當下你才會考慮到可能情況，之前有寫工廠factory這就可以派上用場，每個function名稱都要一目瞭然，名字再長也沒關係，e.g.,only_logged_in_user_can_edit_profile
`tests\Unit`資料夾底下建立我們功能檔案，我們通常會對一個model作撰寫，unit與feature的差別是，feature包含了對使用者UI互動的測試，例如網頁會轉跳到哪裡的驗證

## 常用的方法
```php
//這一行會幫我們migrate，爾且清空
use RefreshDatabase;
//所有function運行前都會先做放在裡面的邏輯，
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
//laravel內建http錯誤碼處理關掉，看真正的報錯
$this->withoutExceptionHanding();
//忽略Event
Event::fake();
$this->actingAs(factory(User::class)->create(
    'email'=>'admin@admin.com'
));
$this->post('/add',[
    'name'=>'test user',
    'email'=>'test@test.com'
]);
$this->assertCount(1,User::all());

//an_email_is_required
$response = $this->post('add',array_merge($this->data(),[
    'email'=>''
]));


```



# 使用情境
## 圖片測試
Q:How to create a phpUnit test for images?
A:I'd prefer not to test such things
https://stackoverflow.com/questions/39041876/how-to-create-a-phpunit-test-for-images

