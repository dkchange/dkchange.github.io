---
title: 使用mysql
date: 2020-03-09 11:08:30
tags:
categories:
---

# 前言

# 環境設定

# 遭遇案例

1. Access denied for user 'root'@'localhost'
   你只能藉由 sudo mysql -p 來登入資料庫是一件很麻煩的事，你的程式也需要權限來訪問資料庫
   https://blog.csdn.net/zhouyingge1104/article/details/85254895
   https://www.opencli.com/mysql/reset-mysql-mariadb-root-password

2. Display positive and negative values in different column

https://www.c-sharpcorner.com/forums/display-positive-and-negative-values-in-different-column

3. MYSQL 查找所有的父级或子级
   https://segmentfault.com/a/1190000007531328

# 分表

https://kkc.github.io/2017/07/07/mysql-partitioning/

# 效能問題

- server 硬體
  - cpu 要看當前的 mysql 支不支援多核心處理同一語句
  - 那如果一個核心只能處理一句，那 n 個核心就可以處理 n 句 sql
  - 在高併發的情況下，多核心會比高頻的 cpu 重要
  - 64 位元的 cpu 不要用 32 位元版本的軟體
  - 大記憶體（內存），高頻率記憶體
  - 傳統硬碟，高容量，高傳輸速度，高轉速，物理尺寸小
  - RAID 多個小硬碟組成一個大硬碟
    - RAID0 讀快寫快，沒有修復能力
    - RAID1 讀快寫慢，有鏡像備份
    - RAID5 讀快寫中，有備份
    - RAID10 讀快寫快，有備份
  - SSD 受限 sata 界面傳輸速度 支援 RAID
  - PCIESSD 貴，要裝驅動，但不受限於 sata
  - SAN NAS 網路儲存 不如 RAID，只適合做數據備份
  - 網路，高性能交換機，對多張網卡進行綁定，用以增加頻寬
- server 系統
  - windlow 系統下不分大小寫，需要做參數設定讓其能區分
  - freebsd
  - solaris
  - linux 挑選 server 版的
- db 儲存引擎選擇
  - MyISAM:5.5 以前的默認引擎，不支持 transation，支援表級鎖，可壓縮表，適合只讀的應用
  - InnoDB:5.5 以後的默認引擎，5.6 默認使用獨立表空間儲存數據，支持 transation,支援行級鎖，事務 acid 特性，5.7 之後支援全文索引，空間函數
  - csv:非二制文件，是以儲存文本的方式，所以可以直接查看，就是 excel 的文件來當一個表，不支援索引，不支援 null
- 參數設定
  - centos
    - 內核相關參數/etc/sysctl.conf
- db 結構設計和 sql 語句

QPS：Queries Per Second 意思是“每秒查詢率” 每秒可以查詢幾句 sql
TPS：是 TransactionsPerSecond 的縮寫，也就是事務數/秒
併發量：高併發要調高 mysql 連接數 max_connections 默認 100
cpu 使用率
硬碟 io
網卡流量：
避免使用 select \*

# 大表遇到的問題

主從延遲
修改表結構時間長，會鎖表

解決方法：
分庫分表
歷史數據歸檔

# transation

## 原子性 atomicity

一筆交易最小的單位，失敗就全部回滾

## 一致性(consistency)

轉帳前轉帳後，總金額須一樣

## 隔離性

- 未提交讀
- 已提交讀（大多數 rmdb 的預設）
- 可重複讀
- 可串行化

## 持久性

資料不會不見

> show variables like '%iso%'

# 大 transation 遇到的問題

鎖的表太多會造成阻塞
rollback 時間長
容易造成主從延遲

# transation 新手問題
在MySQL的InnoDB中，预设的Tansaction isolation level 为REPEATABLE READ（可重读），所以事務的操作除了可以維持原子性，提供的鎖也可以避免髒讀，不重複的讀取

1.其他 proccess 去 select 在 transaction 中變動的資料 會是變動前還是後？

要看其他 proccess 怎麼去 select，如果下 nolock 就可以查到未 commit 的資料，而正常情況下因為 update 的資料會使用獨佔鎖，所以其他 proccess 會被鎖住，必需等待交易完成後才能查詢資料。

2.使用 command 直接 kill 正在 sleep 的 proccess 是會自動rollback嗎？

會，未 commit 的資料，連線中斷或資料庫當機，都會自動 rollback，這樣才能確保資料是正確可靠的。
https://ithelp.ithome.com.tw/questions/10192987?sc=rss.qu
https://stackoverflow.com/questions/22846438/why-does-select-for-update-works-only-within-a-transaction

https://www.cnblogs.com/houweijian/p/5869243.html

https://segmentfault.com/a/1190000014133576

https://segmentfault.com/a/1190000022732257

# 案例
解决方案：
- 把SELECT和UPDATE合成一条SQL
- 用一个事务来包裹上面的SELECT+UPDATE操作
- 乐观锁，类CAS机制
[MySQL中SELECT+UPDATE并发更新问题](https://blog.csdn.net/dainandainan1/article/details/109120038)

# 影響數據庫的效能
- 表的設計合理化(符合 3NF)
- 添加適當的索引
 a. 普通索引
 b. 主鍵索引
 c. 唯一索引
 d. 全文索引
 e. 空間索引
 - 分表
 a. 水平分割
 b. 垂直分割
 - 讀寫分離
 - 存儲過程SP 因為不需要再編譯
 - 配置最大併發數
 - 硬體升級
 - 定期清除數據，進行碎片整理

 # 主從同步
 https://cloud.tencent.com/developer/article/1832929

 # 比較少用的sql語句

 sql find_in_set() vs match() against()

 force index

 explain
 https://www.cnblogs.com/acm-bingzi/p/mysqlExplain.html
 https://smilenicky.blog.csdn.net/article/details/100853310


 ## 查找資料庫裡面的欄位名稱
 select column_name,table_name from informateion_schema.columns where column_name like '%ip%';

# 主從複製-單表複製
https://blog.csdn.net/weixin_67857994/article/details/123546958?spm=1001.2101.3001.6650.7&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-7-123546958-blog-113947355.pc_relevant_aa&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-7-123546958-blog-113947355.pc_relevant_aa&utm_relevant_index=10
主從複製單表或多個表

# 區分主庫從庫
http://www.itpub.net/thread-1737186-1-1.html

slave会有relay-log日志文件
or

有没有master.info的信息，有就是从库
or
show slave status
show master status
or
> show global status like 'Slave_running';  


# mysql in not in 優化

用了exist變快
https://blog.csdn.net/gua___gua/article/details/47401621
用了exist變慢
https://blog.csdn.net/fly910905/article/details/78288685

 # db 塞假資料
產生測試資料(假資料)
https://ithelp.ithome.com.tw/articles/10147387?sc=pt

# log before cascade delete
https://stackoverflow.com/questions/11496459/can-i-use-a-procedure-to-store-history-log-before-on-delete-cascade

# opratoin log in mysql
https://stackoverflow.com/questions/303994/log-all-queries-in-mysql

# mysql單表同步

1. 如果你只是要臨時複製AB表的資料，
語法就可以達成:
其中主鍵要對應好，不然會有資料遺失的問題
insert into A資料庫名稱.A資料表 select * from B資料庫名稱.B資料表
where not exists(select * from A資料庫名稱.A資料表 where A資料庫名稱.A資料表.ID = B資料庫名稱.B資料表.ID)

2. 創建觸發器
trigger

```sql
use test1

DELIMITER //

CREATE TRIGGER Pee_insert_host AFTER INSERT ON host FOR EACH ROW BEGIN INSERT INTO test2.host VALUES (new.id,new.host,new.port,new.user,new.pwd); END;//

CREATE TRIGGER Pee_delete_host AFTER DELETE ON host FOR EACH ROW BEGIN DELETE from test2.host where test2.host.id=old.id; END;//

CREATE TRIGGER Pee_update_host AFTER UPDATE ON host FOR EACH ROW BEGIN UPDATE test2.host SET host=new.host,port=new.port,user=new.user,pwd=new.pwd WHERE test2.host.id=new.id; END;//

// 查看触发器：
SELECT * FROM information_schema.`TRIGGERS`;

// 删除触发器：
DROP TRIGGER TRIGGER_NAME;
```

原文：https://blog.csdn.net/weixin_39958019/article/details/113634347

3. 用mysql 同步功能
google 搜尋 mysql主从只同步部分库或表

原文:[mysql主从只同步部分库或表](https://www.cnblogs.com/weifeng1463/p/8662241.html)
master端：
binlog-do-db      二进制日志记录的数据库（多数据库用逗号，隔开）
binlog-ignore-db 二进制日志中忽略数据库 （多数据库用逗号，隔开）

slave端
replicate-do-db    设定需要复制的数据库（多数据库使用逗号，隔开）
replicate-ignore-db 设定需要忽略的复制数据库 （多数据库使用逗号，隔开）
replicate-do-table  设定需要复制的表
replicate-ignore-table 设定需要忽略的复制表 
replicate-wild-do-table 同replication-do-table功能一样，但是可以通配符
replicate-wild-ignore-table 同replication-ignore-table功能一样，但是可以加通配符


原文:[MySQL 主从同步：同步相关信息的存储及查询](https://blog.51cto.com/yueyinsha/5243070)

# 主從同步故障排除

https://blog.longwin.com.tw/2013/09/mysql-replication-error-1236-fix-2013/
https://aws.amazon.com/cn/premiumsupport/knowledge-center/dms-cdc-error-1236-msql/


# foreignkey 命名規則
會注意到這個的原因是因為，資料庫報錯1022
https://stackoverflow.com/questions/15014592/mysql-error-1022-when-creating-table
https://stackoverflow.com/questions/199498/foreign-key-naming-scheme

# 先join再查詢，先查詢再join
其實我們不知道DB的行為會是怎樣，所以一切都需要驗證之後才知道
https://blog.csdn.net/qq_15329947/article/details/96482433



