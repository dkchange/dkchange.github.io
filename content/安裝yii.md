---
title: 使用yii2
date: 2020-04-06 14:51:59
tags:
categories:
  - 後端路線
  - framework
  - Yii2
---

# 前言

你需要知道的 Yii：

> 參考：[Yii 框架介紹 - Easy Yii](https://easy-yii.github.io/2017/01/09/know-PartFive/)

PHP 目前最流行的 2 大框架，[[Laravel]] 和 [[Yii]] 的對比：
隨著新興的微服務/納米服務體系結構以及對具有獨立維護/部署的小型服務的需求，因此不再建議需要做很多事情的大型框架，所以最近框架也將一些非必要的功能拆出。
兩大框架用更新速度來看，大概就是青少年對上中年男子...

工作需求開始學 Yii：

> 參考：[Yii-China](https://www.yii-china.com/)
> 文檔可能我還不習慣，頭看得很痛：[Yii2 官方文檔](https://yii2-framework.readthedocs.io/en/stable/)

# 安裝 Yii

懶人包影片：[YouTube 教學](https://youtu.be/sRJ6GYiCwkI)

> 參考：[Fancyecommerce 教學](http://www.fancyecommerce.com/2016/05/18/yii2-%E5%88%9D%E5%A7%8B%E5%8C%96%E7%9A%84bootstrap%E8%BF%87%E7%A8%8B-%E5%BC%95%E5%AF%BC/)

看起來 [[Yii]] 分兩種版本，一種是有 init 檔的下面會提到：

```bash
php composer.phar create-project yiisoft/yii2-app-basic basic 2.0.15
php composer.phar create-project yiisoft/yii2-app-advanced advanced 2.0.15
```

這兩個專案的資料夾結構完全不同，切記。一開始吃了悶虧在研究，所以公司的專案並沒有因為 [[Docker]] 就把 [[Yii]] 的資料夾結構做修改。

# Yii 資料夾結構

> 參考：[LearnKu 文章](https://learnku.com/articles/31435)

## Yii Basic 大致介紹

| 目錄       | 說明                     |
| ---------- | ------------------------ |
| assets     | 前端打包用               |
| commands   | console 命令用           |
| config     | 設定檔                   |
| controller | 控制器                   |
| mail       | email templates          |
| model      | 模型                     |
| runtime    | 框架產生的檔案           |
| tests      | 測試                     |
| vagrant    | vagrant container system |
| vendor     | all package              |
| view       | 視圖                     |
| web        | 入口檔案                 |
| widget     | 使用者界面的元件         |

## Yii Advanced

公司的專案大多採用此架構，進而分出其他的專案例如 API 等等。

如何新增 API 專案：[Csdn 教學](https://blog.csdn.net/post_mans/article/details/72876763)

# 設置 yii

all the key in config file are the public properties of the application，詳見：[Yii 應用程式設定](https://www.yiifrframework.com/doc/guide/2.0/en/structure-applications)
id
name
language
defaultRoute=>'site/login' 預設入口
layout=>'main view 的預設模板
layoutpath=> 改變 layout 路徑
on beforeRequest 常數事件
all the components are singleton objects
component=>你看得到的預設設定都是核心元件
像是
request cache user errorHandler db log mailer ...
那有些元件會沒有設定 class 代表，這個元件的 class 是由核心決定傳入的 key 是什，就用哪個 class，詳見：[Yii 元件設定影片](https://youtu.be/iB-CllQVZTg)
再來就是註冊完 component 之後，可用 yii::＄ app->componentname 來存取，如果沒有存取，就不會實例化這個 class，
'test'=>[
class=>'app/component/test'
]
OR
'test'=>function(){
return new app/component/test();
}
也可以和 boostrap 這個屬性做搭配，註冊在這裡面的一定會被實例化

## uncomment the code of url manager

記得記 log 的資料夾和前端 assets 的寫入權限要打開，如果你的專案權限沒設定的話，就打開成 777 吧

## yii basic 遇到的問題

cookieValidationKey must be configured with a secret key，詳見：[Stack Overflow 討論](https://stackoverflow.com/questions/38824006/error-when-installed-yii2)
個人覺得英文版的參考文檔要比簡中版本的易懂，所以如果看不懂簡中的，可以試著看看英文版，詳見：[Yii 設定教學](https://www.yiifrframework.com/doc/guide/2.0/en/concept-configurations)

# 開啟 debug 工具

## 遭遇問題

config 的 allowIPs 要設定好
例如：

```php
'bootstrap' => ['debug'],
'modules' => [
    'debug' => [
        'class' => 'yii\debug\Module',
        'allowedIPs' => ['127.0.0.1', '::1','172.19.*'],
    ]
]

```

runtime 的資料夾權限記得改為可寫入

# yii 知識點

Yii2 use "new" everywhere to create models.
Why Yii2 use "new" everywhere while pushing their "Dependency Injection Container" feature that is not compatible with "new"
Performance mainly.，詳見：[Yii 論壇討論](https://forum.yiifrframework.com/t/new-vs-yii-createobject/86677)

# 安裝完之後

init 腳本：[GitHub yii2-app-advanced](https://github.com/yiisoft/yii2-app-advanced/blob/master/init)

## 遭遇問題

to solve problem like
Unable to write the file '/var/www/yiitest/models/Country.php'
php init
之後產生出來的檔案 yii 都歸在 ignore
然後實驗證明官方的 php init 並不會幫你改其他資料夾的權限，但是 runtime 的資料夾權限會幫你改，才能記 log

chmod -R 777 -R .
為何经由 gii 安装出来的档案 owner 会是 apache,原因是除了 nginx.conf 档要修改以外,php-fpm 底下的 www.conf 档也要改
user = nginx
group = www-data

php yii migrate
跟 laravel 一样

sudo chown -R myuser:mygroup myfolder

# 运行 yii

yii 分两入口，一部份是 web，另一部份是 console
所以设定档也分成 common 和 console
再继续细分為 local 后缀和非真正生产环境的设定档

# 运行错误

运行时可能有遭遇到的错误
Fatal error: Cannot use 'Object' as class name as it is reserved，詳見：[GitHub Issue](https://github.com/yiisoft/yii2/issues/14823)

# yii 生命周期

Yii 框架生命週期詳解：[掘金文章](https://juejin.im/post/5d51056be51d4561c02a250b)

# yii console

This is [[Yii]] version 2.0.15.1.
The following commands are available:

- asset Allows you to combine and compress your JavaScript and CSS files.
- cache Allows you to flush cache.
- fixture Manages fixture data loading and unloading.
- gii This is the command line version of [[Gii]] - a code generator.
- help Provides help information about console commands.
- message Extracts messages to be translated from source files.
- migrate Manages application migrations.
- serve Runs PHP built-in web server.
  那記得每個指令最好要用之前都先透過 yii help XXXX，會有詳細的說明，以及列出其他的指令

No command 'yii' found,
./yii migrate/create user

or
add it to the PATH variable in .bashrc:
vi ~/.bashrc

export PATH="/var/www/yii/framework:\$PATH"
export PATH="$HOME/.composer/vendor/bin:bin:$PATH"

就像 nodejs
echo 'export PATH=$PATH:/usr/local/bin' >> $HOME/.bashrc

In my case (with ZSH), it was: echo 'export PATH=$PATH:/usr/local/bin' >>$HOME/.zshrc ...and then I ran: source ~/.zshrc

or

create symlink in /usr/bin (UBUNTU)
go to /usr/bin and
ln -s /data/lib/php/yii/framework/yiic yiic

## 建立 controller

yii gii/controller --controllerClass = app\controllers\TestController

或者使用 web 界面
backend/config/main.php

```php
   'components' => [
       'urlManager' => [
           'enablePrettyUrl' => true,
           'showScriptName' => false,
           'rules' => [
           ],
       ],
   ],
```

http://localhost:8088/?r=gii（本地開發用 Gii 入口）
或是
url 美化後之後 gii 會提到
http://localhost:8088/gii（本地開發用 Gii 入口，啟用 Pretty URL 後）

# yii 的資料夾結構

Yii 的資料夾結構除了 gii 之類的指令會生成檔案，大致上就沒什麼變，大部分都要手動新增，例如你需要手動新增 fixture

# yii Validation

[[Yii]] 欄位驗證教學：[Yii2 官方文檔](https://yii2-framework.readthedocs.io/en/stable/guide/input-validation/)

Yii 在做欄位驗證的時候，
可以對整個表單做驗證，
$form->validate()
可以對單一欄位做驗證
$form->validate(['field_name'])

那如果需要特別指定（Ad hoc）某個驗證規則
$validator = new yii\validators\EmailValidator();

那傳統 form 表單傳過來的資料都是字串，所以如果是數字，PHP 會自動進行轉換
所以 rule integer 就算你是字串他也會給你過，那萬一你是對型別有要求的 json，那傳統表單的驗證規則和 json 的就不能共用，詳見：[NumberValidator 文檔](https://www.yiifrframework.com/doc/api/2.0/yii-validators-numbervalidator)
請加上 integerOnly=>true

Yii2 Cookbook：[多屬性驗證](https://flamerecca.gitbooks.io/yii-2-0-cookbook-yii2-gitbook-cookbook/forms-validator-multiple-attributes.html)

## 有關欄位個別更新

因為欄位有時需要同意支 API 可更新個別單個欄位，此時要自己在家邏輯看看欄位是否為 null
但記得 trim 過後的欄位一定是空字串，此時你會無從判斷使用者是否有傳值過來，所以如果允許 null 的欄位建議就不要再加 trim 規則，
如果要的話，就變成只能判斷是否為空字串，就要加上 empty 判斷，讓你的欄位不能是空字串，但通常允許 null 的欄位都可以是空字串，在此建議可以不用加 trim 以免麻煩

## safe

Yii safe 属性：[YiiChina 討論](https://www.yiichina.com/question/2093)

## 如何自訂驗證和參數

Yii 自訂驗證器：[GitHub Issue](https://github.com/yiisoft/yii2/issues/1174)

传参数要用关键字**params** 否则会报错

```php
    public function rules()
    {
        return [
            ['test', 'validateTest', 'params' => ['test' => 'value']],
        ];
    }
```

## 密碼驗證

generatePasswordHash
Yii 的加密函數和一般的 md5 hash1 不一樣，即便輸入相同也會有不同的輸出
所以必須透過 Yii 的 validatePassword 來驗證
這點是蠻特別的

# yii 中学习到的原则

1. 要考虑维护性
   scenarios ,最好设定常数,不要直接用字串，这样档案之前才可以互相参考和提示
2. 要划分好职责,就像一般 mvc 会遇到的问题一样，有人会建议在画个 repository

- controller的原則，超過10行就是臃腫的controller
- controller放不會重複的code

3. 写 code 的时候也要对 function 划分好职责，不需要做的而外判断就不要加上去，就像工程师对 function 参数的输入在外面验证过后，就不要在验证一次了，要相信写 code 的人的输入，至于如果是使用者的输入，就一定要验证，要分清楚工程师和使用者的输入

# yii scenarios

## Rule

[[Yii]] Scenarios 教學：[Yii-China](https://www.yii-china.com/post/detail/40.html)

## 欄位驗證

像在做 API 和後台欄位會有不同的客製化需求，主體的 model 不變，但是驗證規則繼承下來之後，多半都是需要重寫的。若不想要重寫，建議設置 **on** scenarios，
尤其是 required 規則一繼承下來大家都吃的到，這樣要運用就不是很好運用，非必填變成一定要填，除非你能確定該規則換情況都通用，例如密碼格式、長度，這些可以共用就不用設置 scenarios。
另外就是例如 unique 規則會去訪問 DB，所以驗證的 form 除了一定要繼承自原本的 model 以外，一定要加上 scenarios，不然查詢功能還要驗證參數的 username 是否唯一這是什麼

# yii 調適

show raw sql query
\$query->createCommand()->getRawSql()
詳見：[顯示原始 SQL](https://www.yiifrframework.com/wiki/857/show-raw-sql-query)
或者記下 log：[Yii 論壇討論](https://forum.yiifrframework.com/t/is-there-a-log-of-sql-queries/27533)

# gii

如果开启了 pretty URLs, 则这样访问:
http://localhost/path/to/index.php/gii（Gii 入口）

gii forbidden (#403)
如果从除 localhost 之外的 IP 地址访问 gii ，访问将被默认拒绝。 要规避该默认值，则需将允许的 IP 地址添加到配置中：
yiitest/config/web.php
那如果是進階版， main.php

```php
'modules' => [
'gii' => [
'class' => 'yii\gii\Module',
//'allowedIPs' => ['127.0.0.1', '::1', '192.168.1.*', 'XXX.XXX.XXX.XXX'] // adjust this to your needs
'allowedIPs' => YII_DEBUG ? ['*'] : ['127.0.0.1', '::1'],

    ],
```

Yii Gii 模組：[官方文檔](https://www.yiifrframework.com/extension/yiisoft/yii2-gii/doc/guide/2.1/zh-cn/installation)

# 啟動 server 放文件給同事看

php -S 0.0.0.0:10000 -t mapi/apidoc

PHP 的 built-in web server 有个 -t 参数可以指定 document root 的路径的，比如这里就可以写成：
php -S localhost:8080 -t web
这样就能省掉 URL 中的 /web/index.php 这部分了。

# ERROR : (2006, 'MySQL server has gone away'

是因為設定檔忘記將 mysql:改成 pgsql:

# phpstorm shortcut duplicate lines

(you'd use ctrl+d, but first you have to select many (how? I think they wanted ctrl+w to be used for this))

# 取資料

```php
$query = GetGameRecord::find()
    ->select([
        'game_record.*',

'game_kind.kind_name'
])
->joinWith('gameKind')
->Where(['board_id' => $pageForm->board_id])
->orderBy('settle_time desc');
```

# yii 分頁

這是 Yii ORM 內建的子查詢，如果不想要子查詢那麼多
可以在下 SQL 的時候作 join 像上面一樣

```php
    public function fields()
    {
        $fields = parent::fields();
        $fields['kind_name'] = function () {
            return $this->gameKind ? $this->gameKind->kind_name : null;
        };
        $fields['server_name'] = function () {
            return $this->gameServer ? $this->gameServer->name : null;
        };
        return $fields;
    }


     $firstQuery= Agent::find()
            ->where(['parent_id' => $this->agent->id])
            ->andFilterWhere([
                'id' => $pageForm->agent_id,
                'nickname' => $pageForm->nickname,
                'username' => $pageForm->agent_username,
            ]);

        $secondQuery = GameRecord::find()
            ->Where(['>=', 'settle_time', ($pageForm->start_time != null) ? $pageForm->start_time : null])
            ->andWhere(['<=', 'settle_time', ($pageForm->end_time != null) ? ($pageForm->end_time + 86399) : null])
            ->andFilterWhere([
                'kind_id' => $pageForm->kind_id,
            ]);


        $query = Agent::find()
            ->select([
                't1.id',
                't1.username',
                't1.nickname',
                't1.parent_rate',
                't1.score',
                'bet' => new Expression('COALESCE(SUM(valid_bet), 0)'),
                'count' => new Expression('COUNT(board_id)'),
                'total' => new Expression('COALESCE(SUM(revenue), 0)'),
            ])
            ->from(['t1' => $firstQuery])
            ->leftJoin(['t2' => $secondQuery], 't2.agent_id = t1.id')
            ->orderBy('t1.id desc')
            ->groupBy(['t1.id', 't2.agent_id'])
            ->asArray();
        echo $query->createCommand()->getRawSql();




                $subQuery = MemberLoginLog::find()->select([
            'username',
            "MAX(created_at) last_time_login",
        ])->groupBy('username');



        $query = Member::find()->select([
            'member.username',
            ])
            ->leftJoin(['last_login'=>(new \yii\db\Query())
                ->from(['t1'=> $subQuery])
                ->innerJoin([MemberLoginLog::tableName()],'t1.username=member_login_log.username AND t1.last_time_login=member_login_log.created_at')
            ],'last_login.username=member.username')
            ->andWhere(['member.agent_id' => $this->agent->id])->orderBy('created_at desc')
            ->groupBy('member.username');
```

# migrate/fresh

資料表在 migration 的時候會合叫加％符號，是因為之後再加前綴才會加到
实验后证实，经过 partition 的表，在 fresh 时会失败

## yii 連接 tsql

Yii 連接 T-SQL：[Yii 論壇](https://forum.yiifrframework.com/t/odbc-support/42598/6)

# yii 多個表 join

Yii 多表 JOIN：[Yii-China](https://www.yii-china.com/topic/detail/110)

# thousands of single SELECTs

Yii ActiveRecord N+1 問題：[Stack Overflow](https://stackoverflow.com/questions/30195428/why-is-yii2s-activerecord-using-lots-of-single-selects-instead-of-joins)

# yii 格式化輸出

Yii 格式化輸出：[GitHub Issue](https://github.com/yiisoft/yii2/issues/10927)

# 用 join 進來的表格做排序

Yii DataProvider 排序：[Yii 論壇](https://forum.yiifrframework.com/t/sort-dataprovider-with-count-on-related-field/82810)

# 如何分割 log 檔

[[PHP]]/Yii 日誌設定：[腳本之家](https://www.jb51.net/article/81094.htm)

```php
        'log' => [
            'targets' => [
                [
                    'class' => 'yii\log\FileTarget',
                    'logFile' => '@runtime/logs/' . date('Y') . '/' . date('m') . '/' . date('d') . '/app.log',
                ],
            ],
        ],
```

# Controller

controller 介於 View 和 model 之間做溝通的橋樑，
router 和 function 的名字做對應，有大小寫和 dash 的規則
public $defaultAction = 'XXX';
可以改變 router 根目錄/為 index 的預設
public $layout = 'main';
可以改變 view 的 layout 模版
public \$enableCsrfValidation = false;
關掉 csrf 驗證，通常不建議這麼做
function actionXXX 的參數可以作為 queryString 參數的接口
Yii Controller 影片教學：[YouTube](https://youtu.be/hQr0ce2OvmA)

事件註冊：
以 on beforeAction 為例，註冊在 config 裡面的會先於個別 controller,

```php
'on beforeAction'=>function(){

}

public function beforeAction(){

}

先於註冊在 controller 的事件例如：
Yii::\$app->controller->on('controller::EVENT_BRFORE_ACTION',function(){})
```

## render

```php
$this->render('about');
$this->renderPartial();//render html
$this->renderFile();
$this->renderAajx();
```

# view

## render 部份內容

\$this->render('footer');

## 傳資料

在 controller 你可以
$this->render('path',['data'=> 'mydata');
$this->view->params['data'];
在 view 這樣收
$data
$this->params['data'] or Yii::\$app->view->params['data']

## 找出 controller

\$this->context->id

## 其他標籤

$this->title
$this->registerMetaTag(['name'=>'keyword','content'=>'yii2 , framwork'])

## 小技巧

在 view 頁面,加上
/\*_ @var \yii\web\View \$this _/
PHPstorm 就會知道這個頁面的\$this 是 view 的實例

# model

在 controller 如果你要存取 model 的屬性的話
$model->attributeName
$model['attributeName']
都可行
那如果要大量賦值的話
$model->attributes = $post;
那記得一定要經過\$model->validate()

# request

在 controller 我們可以
Yill::$app->request->get();
Yill::$app->request->get('id',0);
或是經由 function actionXXX($id=0)的參數
Yill::$app->request->getBodyParams()
用在你希望收到 put 請求或是 json 格式的參數時
阿如果是 json
設定檔頁要做設定

```php
'component'=>[
    'request'=>[
        'parser'=>\yii\web\JsonParser::class
    ]
]
```

# response

Yii Response 影片教學：[YouTube](https://youtu.be/omqyY94WVos)
當你在 actionXXX 回傳東西的時候就相當於

```php
function actionXXX(){
return 'hello world'
}

Yii::\$app->response->content = 'hello world'
```

## statusCode

```php
Yii::\$app->response->statusCode = '404'
// 或者
// https://www.yiiframework.com/doc/guide/2.0/en/runtime-responses
throw new \yii\web\HttpException(404)
throw new yii\web\NotFoundHttpException();
```

## 改變回傳格式

```php
Yii:$app->response->data = [
    'name' => '',
    'age'=> 5
];
Yii:$app->response->format = Response::FORMAT_XML
Yii:\$app->response->format = Response::FORMAT_JSON
```

## 重導畫面

```php
或在 controller
Yii::$app->response->redirect('about',301);
 $this->redirect('about');
在 controller 外
Yii::\$app->response->redirect('about',301)->send();
```

## 傳檔案

```php
Yii::$app->reponse->sendContentAsFile('hello world','test.text')
Yii::$app->reponse->sendFile('filepath','test.text')
Yii::\$app->reponse->sendStreamsFile()//這個在傳送大檔的時候，特別重要，才不會報出記憶體不夠的錯
```

# layout

Yii Layout 影片教學：[YouTube](https://youtu.be/69zKqHrU5-Y)

在 layout 頁面
\$this->beginPage()會呼叫 on beforePageBegin 事件

# widget

UI 元件的製作

# alias

Yii::setAlias('@home','/home');
@符號如果忘記加，yii 會幫你加

# asset bundle

Yii Asset Bundle 影片教學：[YouTube](https://youtu.be/eJBQeRocRac)
在你不需要內建的 css 和 js 的時候，你需要自訂義

# dao

一般來說 yii 提供
new Connection([
'dsn'=>'',
'username'=>'root',
'password'=>'password'
]);
來建立資料庫連線，但不推薦這樣做
應該用官方的設定 component 的方式，component 都是 singleton,省資源
再來存取這個物件

```php
   Yii::$app->db->createCommand('select * from user where id = :id')->query();
   Yii::$app->db->createCommand()->bindValues(['id'=>$id])->query(); //用於大量賦值
   Yii::$app->db->createCommand()->bindParams('id',$id)->query(); //注意bindParams第二個參數是吃位址不是吃值

   Yii::$app->db->createCommand()->insert('user',[])->execute();
   Yii::$app->db->createCommand()->batchInsert('user',['username','age'],[$dataArr])->execute();

   Yii::$app->db->createCommand()->update('user',$data,['id'=>5])->execute();
   Yii::$app->db->createCommand()->delete('user',['id'=>5])->execute();

   Yii::$app->db->createCommand()->upsert('user',$data,['id'=>5])->execute();

```

## backtick

框架會自訂幫我們加\`
但是有的時候框架會因為語句太複雜而加錯地方

```php
$db->createCommand('SELECT IFFULL(email,username) from user')->execute();
出來的語句會有錯
所以必須要
$db->createCommand('SELECT IFFULL([[email]],[[username]]) from {{user}}')->execute();
```

# Active Record

我們的 model 通常都會繼承 [[ActiveRecord]]
Model::findOne($id);
update insert用
$model->save();
delete 用
\$model->delete();

# query bulider

(new Query())->select([])->from()->all();
阿因為 ActiveRecord extends ActiveQuery and ActiveQuery extends Query
所以等於
Model::find()->select([])->from->all();
又因為是 ActiveRecord 物件可以省下填寫表格名稱變成
Model::find()->all();

# 設定物件

yii 提供了讓你創建物件的方法
例如
Yii 設定物件影片教學：[YouTube](https://youtu.be/lQaCY8Zza2U)

```php
Yii::Createobject([
    'class'=>Myclass::class,
    'otherproperty'=>10,
    'otherproperty2'=>20
]);
// 等同於
$object = new Myclass();
Yii::configure($object,[
    'otherproperty'=>10,
    'otherproperty2'=>20
]);
// 就像設定檔的給的陣列一樣

// 我們也可以在設定檔設定容器
'container' => [
    'definitions'=>[
        'NameSpace/Myclass'=>[
            'otherproperty'=>10,
            'otherproperty2'=>20
        ]
    ]
]

// 然後
$object = Yii::Createobject([
    'class'=>Myclass::class,
]);
// 就可以發現物件的設定值已經改變
var_dump($object);
或者
Yii::$container->set(Myclass::class,[
    'otherProperty' => 10
])


```

# restAPI

Yii REST API 影片教學：[YouTube](https://youtu.be/XyHHMvRt6Cw)

yii 支援 restapi，只要做幾個設定，就能輕鬆做出 restful 的 api

controller extends yii/rest/ActiveController
指定 model
\$modelClass = modelClass::class;

```php
'request'=>[
    'parser'=>[
        'aplication/json'=>JsonParser::class
    ]
]

'urlManager'=>[
    'enablePrettyUrl'=>true,
    'showScriptName'=>false,
    'rules'=>[
        [
         'class'=>UrlRute::class,
         'controller'=> ['contorllerName','controllerName2']
        ]
    ]
]
```

# yii 框架 日誌設定

PHP/Yii 日誌設定：[腳本之家](https://www.jb51.net/article/81094.htm)

# 當你的 php 是 5.2x 不支援 sha256

OpenSSL SHA256 需求：[Stack Overflow](https://stackoverflow.com/questions/10524198/what-version-of-openssl-is-needed-to-sign-with-sha256withrsaencryption)
此篇像上帝一般，生產環境動不了只好動 code
