---
title: yii使用codeception
date: 2020-05-08 09:09:35
tags:
categories:
---

# 前言

[中文 Codeception 的基本設定](https://juejin.im/entry/588049612f301e00697dcb6a)
[原文 Codeception 的基本設定](https://github.com/yiisoft/yii2-app-advanced/blob/master/docs/guide/start-testing.md)
開始之前一定要跟教學建一個測試專用的資料庫，因為測試用的資料都會被清除，除非你想要內測的資料每次都被清除，否則只好照做
[參考慘案：測試資料都被刪光了](https://segmentfault.com/a/1190000008157520?utm_source=tag-newest)
https://segmentfault.com/u/zebrayoung/articles?sort=vote

Codeception 是一套獨立的測試框架，可跟其它 PHP 框架做搭配，在這我們和 Yii 做搭配
在這裡推薦 KK 的教學文章，個人覺得中文教學難找，他寫得很詳細，應該說我覺得他寫的比官網還要詳細，是我看錯官網嗎
然後絕對不要再生產環境做測試
[原文官方文檔](https://codeception.com/docs/)
[KK 測試範例](https://github.com/kk8686/xoa)
[KK 中文技術文檔](http://www.kkh86.com/it/codeception/guide-leaning-depends.html)

# 如何編寫測試用例

有什麼比較好的門道，平衡覆蓋率和收益
[如何編寫測試用例](https://www.zhihu.com/question/51558124)（連結已失效）
底下的留言有正交表的相關資訊，那由於學習正交表相關理論需要時間，我目前還是用窮舉的方式來做測試
[接口測試用例到底應該怎麼寫](https://testerhome.com/topics/11744)

## Waterfall Agile DevOps

軟體開發並的管理流程 與其框架 實踐方法

- Waterfall → CMMI
- Agile → Scrum
- Agile → XP
- Agile → Kanban
- Agile → Lean
- Agile → XP → TDD
- DevOps → TDD
  都有軟體開發、測試和部署的階段。
  然而，敏捷流程在這三個階段之後會終止。
  相反，DevOps 包括後續持續的運維。

由以上方法衍生出

- TDD：測試驅動開發（Test-Driven Development）
- BDD：行為驅動開發（Behavior Driven Development）
- ATDD：驗收測試驅動開發（Acceptance Test Driven Development）
- DDD：領域驅動開發（Domain Drive Design）

# 為什麼要寫測試

即使是一個不成功的項目也會有規模增長以及複雜度的增加的
當項目一大，你所寫的 code 可能很多地方都有用到，然而人是健忘的，只有在寫 code 的當下才會清楚這代碼和其他 code 的耦合，那我們能做的就是在當下寫測試，確保修改之後的測試報錯，能讓你回想到寫 code 當下的邏輯，避免真正產品上限之後出現改 A 錯 B 的情形
手動能測試的有限，每次改完都要在手動重測一次很累，而且一定會有測試沒覆蓋到的地方
測試代碼並不能將所有角度都全部測試，因為程式員可能會寫漏一些角度的測試代碼
比如數據搜索的測試，搜索條件組合花樣百出，如果 1000 個商品分類裡有其中 3 個有問題，那為了準確找出這些問題，程式員是不是要寫 1000 種不同商品分類的測試代碼？
這不現實，測試代碼確實無法完美地測試所有問題，但大部分問題已經寫好測試代碼後，一旦修改業務邏輯，你不再需要人工訪問這裡那裡的頁面測試刷新測試刷新

## 哪些類和方法要編寫單元測試？

一切其實都是看性價比，你覺得值了就寫，不值了就別寫
例如 臨時的/少調用的/非核心的類就可以不必為它寫測試方法
你覺得哪些容易出錯，後期修改容易造成誤差的內容嘛，就測那些好了不然你測得再多，有的本身就很穩定，永遠不出錯，你這個測試代碼可能就白寫了，自己學會把握哦

# 代碼覆蓋率（Code coverage）

[基本準則參考 Wiki](https://zh.wikipedia.org/wiki/%E4%BB%A3%E7%A2%BC%E8%A6%86%E8%93%8B%E7%8E%87)

- 函式覆蓋率（Function coverage）：有呼叫到程式中的每一個函式（或副程式）嗎？
- 指令覆蓋率（Statement coverage）：函式中每一行都執行一次了嗎？
- 判斷覆蓋率（Decision coverage）：有 if 的情況，那 true 和 false 這兩個有滿足嗎
- 條件覆蓋率（Condition coverage）：if 裡面所有的條件都有 true 和 false 都考慮到嗎
- 條件/判斷覆蓋率（Condition/decision coverage）：需同時滿足判斷覆蓋率和條件覆蓋率，所有排列組合。
- 其他還有一些準則請參考 Wiki

# 依單位來劃分測試

Codeception 劃分了幾種測試，那測試的劃分也是十分重要，怎樣才算單元？
https://segmentfault.com/q/1010000005160792?utm_source=tag-newest
PHP 原生的對象方法理解為單元，沒有過多層次的調用封裝的
開發者自己將業務串聯起來一起測試的這就是功能測試

## 驗收測試

驗收測試則是主要針對頁面做測試
模擬使用者使用瀏覽器的測試，不限定你後端的語言，你就是個單純的使用者

## 功能測試

和驗受測試很像，沒有模擬使用者的功能測試

## 單元測試

單元測試主要針對函數/方法做測試
根據 SRP 原則，就是一個個 class 做自己的事，測試裡頭的每個 function

## API 測試

# 回歸測試

檢驗軟體原有功能在修改後是否保持完整。

# 設定

文件配置這個我再官網找不到相關說明，請在 KK 的文章找相關訊息
如果你想在本地開發時使用自己特殊的路徑和資料庫信息配置，但是又不想影響其他開發成員，以及在測試機上不使用你自己的配置來運行

dist 配置支持
只要知道「codeception.dist.yml 這個配置文件如果存在的話，會與 codeception.yml 進行合併，並且兩者存在相同配置時會優先使用 codeception.yml 的」

你只要別提交 codeception.yml 到代碼倉庫上就行了

# 断言

所谓的断言就是
我判断这个结果是
我敢断言这个结果是
既然有了 assertTrue 这个几乎万能的断言，为什么还会有那些别的断言呢？--其实是为了提高测试报告的精准性。
例如
$this->assertContains($id, [1,2,3]);
或者
每一个断言方法的最后一个参数都叫做$message，并且是一个可选参数
$this->assertTrue(in_array(\$id, [1, 2, 3]), '无效的 ID'); //这里传了第 2 个参数，一个用于表达错误的消息字符串
让你的测试讯息能清楚些

# 调试

有的时候多个测试方法执行时，会影响到下一个，例如 var_dump()会影响,下一个测试的请求
reponse，他就会说 header 已经送过了，这样什么东西都看不到，此时最好是用 codeception 提供的方法
codecept_debug('abc');
运行的同时也要加上参数，这样就可以 debug 了，不用像 var_dump()写还还要删掉
php E:\codecept.phar run unit --debug
php E:\codecept.phar run unit -d

# 运行机制

我写 api 测试时，只要是 public 的方法都会被运行，命名并没有强制规范，之后再来看看
每一个方法只要裡面有其中一个断言失败，那么整个方法就会中断，直接换测下一个方法
获取当前测试的方法可以用\$this->getName()
https://github.com/Codeception/Codeception/issues/1251
//那因為版本的不同，这边就要看官网写的比较详细
https://codeception.com/docs/07-AdvancedUsage
Codeception\Scenario is also available in Actor classes and StepObjects. You can access it with \$this->getScenario().
\_before 和\_after
可以与其做搭配
如果测试的方法有先后关西，就要透过注解@depends testxxx 来达成，也就是假如 a()失败了 b()就没必要做下去
依赖关系只会在运行整个测试用例的时候生效
每次运行一个测试方法,测试用例都会被重新 new 一次,那么私有属性就变回默认值了

# yii 测试

yii 官方推荐 codecept,然而 codecept 和 test 设定档互相有冲突，所以必须要分开设定
https://github.com/yiisoft/yii2-app-advanced/issues/346

# /目录底下有个 codeception.yml

裡面会 include 需要的测试档案

https://github.com/Codeception/Codeception/issues/5017

我的问题也是 autoload 的问题
因為专案一开始初始化叫做 admin/tests
之后又改成 mapi/tests
PHP Fatal error: Trait 'mapi\tests_generated\FunctionalTesterActions' not found in /var/www/mapi/tests/\_support/FunctionalTester.php on line 21
就解决了

如果你是 yii advenced
https://codeception.com/docs/10-APITesting#REST-API
就近进到应用程式资料夹，再下指令
../vendor/bin/codecept generate:suite api
这样生成的档案才会对应到正确的资料夹
https://www.cnblogs.com/cfYu/p/10531552.html
接著就照上面的设置打指令
../vendor/bin/codecept generate:cest api AgentCest

跑测试
../vendor/bin/codecept run api AgentCest

../vendor/bin/codecept run api AgentCest:testXXX

如果测试的时候遇到，测试的方法不支援，那可能是模组没有载入，我们可以在刚刚建好的 suite 裡面加载
例如我运行
\$I->fail('list contain agent not belong to you');
显示不支援，所以必须在模下面添加 Asserts，至于我怎知道是大写还是小写 ASSERTS,因為我原本填大写找不到，只好去 github 看原码的 class 是大写还是小写...

actor: ApiTester
modules:
enabled: - \mapi\tests\Helper\Api - REST:
depends: Yii2
part: Json - Asserts

# fixtures

使用这功能真的会清空资料库，请小心
https://www.yiichina.com/tutorial/694
https://www.yiiframework.com/doc/guide/2.0/zh-cn/test-fixtures
https://cloud.tencent.com/developer/section/1494451

看解释：
使用场景： 在我们开发或者测试的时候，可能会需要某种特定状态下的数据，比如我想要用户刚下订单还未支付这个状态，那木我们就需要手动进行添加购物车、下订单等惭怍，但是 fixtures 可以直接将这个状态保存下来，如果需要使用这个状态，只要命令行里面加载一下对应的 fixtures 就好了。（待续）

# 当一个方法 code 很多，裡面的 code 没有相依，不想因為其中一个断言失败而中断

use \Codeception\Specify; 这个 Trait
$this->specify('测试目标的说明', $具体的测试工作代码函数)
将 code 包成一块一块的

# 标记哪些方法已经被测试

写过测试的 code 要增加@test 标记,并说明测试用例是哪个命名空间

# 啟用模块

```yml
modules:
enabled:
  - \mapi\tests\Helper\Api
  - REST:
      depends: Yii2
      part: Json
  - Asserts
```

# 测试器

将各个模块合并成了一个类

codecept build
之后就会出现一个 tester 类，那在运行测试时，这个类会备注入进各个方法

# 测试报告

运行测试失败之后会在\_output 产生测试报告

# 使用 faker

失败中....
為了生出测试用的资料，必须要借用这功能来产出一些随机字串和假资料
use faker
https://github.com/yiisoft/yii2/issues/8692

https://github.com/yiisoft/yii2-app-advanced/issues/172

# 使用 feature

清空资料库和载入资料库

记得要

```yml
- Yii2:
    part: [orm, fixtures]
```

切记，使用 feature 会清空资料库，所以一定要在自己测试的资料库执行，也就是说
不建议执行 console 的
loadDATA
yii fixture/load Agent
然后你就会看到非测试的资料空清空之后，加上你指定在\_data 的资料
这不是走测试资料库
会清空掉内测共用资料库，很恐怖
不是我们想要的结果

Set public \$dataFile = **DIR** . '/data/init_login.php'; inside UserFixture

测试指令可行，test alias 路径也可行
参数-d 代表-debug
../vendor/bin/codecept run api AgentCest -d

# dataprovider

dataprovider 可以透過程式的註解將資料帶進我的要測試的 function 裡面，但是有個缺點是他資料只能寫死，因為這個 function 跑在整個測試之前，
所以最後我還是靠迴圈自己來跑動態的資料測試
