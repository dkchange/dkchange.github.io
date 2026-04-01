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
   你只能藉由 sudo [[mysql]] -p 來登入資料庫是一件很麻煩的事，你的程式也需要權限來訪問資料庫
   [重設 MySQL/MariaDB 密碼](https://blog.csdn.net/zhouyingge1104/article/details/85254895)
   [Reset MySQL Root Password（OpenCLI）](https://www.opencli.com/mysql/reset-mysql-mariadb-root-password)

2. Display positive and negative values in different column
   [顯示正負值在不同欄位（C# Corner）](https://www.c-sharpcorner.com/forums/display-positive-and-negative-values-in-different-column)

3. MySQL 查找所有的父級或子級
   [MySQL 遞迴查詢父子級（SegmentFault）](https://segmentfault.com/a/1190000007531328)

# 分表

[MySQL 分表（KKC）](https://kkc.github.io/2017/07/07/mysql-partitioning/)

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
  - wind**o**ws 系統下不分大小寫，需要做參數設定讓其能區分
  - freebsd
  - solaris
  - linux 挑選 server 版的
- db 儲存引擎選擇
  - MyISAM:5.5 以前的默認引擎，不支援 transaction，支援表級鎖，可壓縮表，適合只讀的應用
  - InnoDB:5.5 以後的默認引擎，5.6 預設使用獨立表空間儲存數據，支援 transaction,支援行級鎖，事務 ACID 特性，5.7 之後支援全文索引，空間函數
  - csv:非二制文件，是以儲存文本的方式，所以可以直接查看，就是 excel 的文件來當一個表，不支援索引，不支援 null
- 參數設定
  - centos
    - 內核相關參數/etc/sysctl.conf
- db 結構設計和 sql 語句

QPS：Queries Per Second 意思是“每秒查詢率” 每秒可以查詢幾句 [[SQL]]
TPS：是 TransactionsPerSecond 的縮寫，也就是事務數/秒
併發量：高併發要調高 [[mysql]] 連接數 max_connections 預設 100
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

# Transaction

## 原子性 Atomicity

一筆交易最小的單位，失敗就全部[[回滾]]。

## 一致性（Consistency）

轉帳前轉帳後，總金額須一樣。

## 隔離性

- 未提交讀
- 已提交讀（大多數 [[RDBMS]] 的預設）
- 可重複讀
- 可串行化

## 持久性

資料不會不見

> show variables like '%iso%'

# 大 Transaction 遇到的問題

鎖的表太多會造成阻塞
Rollback 時間長
容易造成主從延遲

# Transaction 新手問題

在 [[MySQL]] 的 [[InnoDB]] 中，預設的 Transaction isolation level 為 REPEATABLE READ（可重讀），所以事務的操作除了可以維持原子性，提供的鎖也可以避免髒讀，不重複的讀取。

1. 其他 process 去 select 在 transaction 中變動的資料，會是變動前還是後？

要看其他 process 怎麼去 select，如果下 nolock 就可以查到未 commit 的資料，而正常情況下因為 update 的資料會使用獨佔鎖，所以其他 process 會被鎖住，必需等待交易完成後才能查詢資料。

2. 使用 command 直接 kill 正在 sleep 的 process 是會自動 rollback 嗎？

會，未 commit 的資料，連線中斷或資料庫當機，都會自動 rollback，這樣才能確保資料是正確可靠的。
[iThelp 討論](https://ithelp.ithome.com.tw/questions/10192987?sc=rss.qu)
[SELECT FOR UPDATE 討論（Stack Overflow）](https://stackoverflow.com/questions/22846438/why-does-select-for-update-works-only-within-a-transaction)

[MySQL 鎖機制（博客園）](https://www.cnblogs.com/houweijian/p/5869243.html)

[MySQL InnoDB 鎖機制（SegmentFault）](https://segmentfault.com/a/1190000014133576)

[死鎖（SegmentFault）](https://segmentfault.com/a/1190000022732257)

# 案例

解決方案：

- 把 SELECT 和 UPDATE 合成一條 [[SQL]]
- 用一個事務來包裹上面的 SELECT+UPDATE 操作
- 樂觀鎖，類 CAS 機制
  [MySQL中SELECT+UPDATE并发更新问题](https://blog.csdn.net/dainandainan1/article/details/109120038)

# 影響數據庫的效能

- 表的設計合理化（符合 3NF）
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
- 存儲過程 SP 因為不需要再編譯
- 配置最大併發數
- 硬體升級
- 定期清除數據，進行碎片整理

# 主從同步

[主從同步（騰訊雲）](https://cloud.tencent.com/developer/article/1832929)

# 比較少用的 SQL 語句

SQL find_in_set() vs match() against()

force index

EXPLAIN
[EXPLAIN 教學（博客園）](https://www.cnblogs.com/acm-bingzi/p/mysqlExplain.html)
[EXPLAIN 分析（Smilenicky）](https://smilenicky.blog.csdn.net/article/details/100853310)

## 查找資料庫裡面的欄位名稱

```sql
select column_name, table_name from information_schema.columns where column_name like '%ip%';
```

# 主從複製-單表複製

[主從複製單表或多個表（部落格）](https://blog.csdn.net/weixin_67857994/article/details/123546958?spm=1001.2101.3001.6650.7&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-7-123546958-blog-113947355.pc_relevant_aa&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-7-123546958-blog-113947355.pc_relevant_aa&utm_relevant_index=10)
主從複製單表或多個表

# 區分主庫從庫

[區分主庫從庫（ITPUB）](http://www.itpub.net/thread-1737186-1-1.html)

Slave 會有 relay-log 日誌文件
or

有沒有 master.info 的信息，有就是從庫
or
show slave status
show master status
or

> show global status like 'Slave_running';

# MySQL IN NOT IN 優化

用了 EXISTS 變快：[IN vs EXISTS（博客園）](https://blog.csdn.net/gua___gua/article/details/47401621)
用了 EXISTS 變慢：[IN vs EXISTS 反向案例（博客園）](https://blog.csdn.net/fly910905/article/details/78288685)

# DB 塞假資料

產生測試資料（假資料）
[iThelp 教學](https://ithelp.ithome.com.tw/articles/10147387?sc=pt)

# Log Before Cascade Delete

[Cascade Delete 記錄（Stack Overflow）](https://stackoverflow.com/questions/11496459/can-i-use-a-procedure-to-store-history-log-before-on-delete-cascade)

# Operation Log in MySQL

[記錄所有查詢（Stack Overflow）](https://stackoverflow.com/questions/303994/log-all-queries-in-mysql)

# MySQL 單表同步

1. 如果你只是要臨時複製 AB 表的資料，語法就可以達成：
   其中主鍵要對應好，不然會有資料遺失的問題

```sql
INSERT INTO A資料庫名稱.A資料表 SELECT * FROM B資料庫名稱.B資料表
WHERE NOT EXISTS(SELECT * FROM A資料庫名稱.A資料表 WHERE A資料庫名稱.A資料表.ID = B資料庫名稱.B資料表.ID)
```

2. 創建觸發器（Trigger）

```sql
USE test1

DELIMITER //

CREATE TRIGGER Pee_insert_host AFTER INSERT ON host FOR EACH ROW BEGIN INSERT INTO test2.host VALUES (new.id, new.host, new.port, new.user, new.pwd); END;//

CREATE TRIGGER Pee_delete_host AFTER DELETE ON host FOR EACH ROW BEGIN DELETE FROM test2.host WHERE test2.host.id = old.id; END;//

CREATE TRIGGER Pee_update_host AFTER UPDATE ON host FOR EACH ROW BEGIN UPDATE test2.host SET host = new.host, port = new.port, user = new.user, pwd = new.pwd WHERE test2.host.id = new.id; END;//

// 查看觸發器：
SELECT * FROM information_schema.`TRIGGERS`;

// 刪除觸發器：
DROP TRIGGER TRIGGER_NAME;
```

原文：[觸發器同步（博客園）](https://blog.csdn.net/weixin_39958019/article/details/113634347)

3. 用 MySQL 同步功能
   Google 搜尋「MySQL 主從只同步部分庫或表」

原文：[MySQL 主從只同步部分庫或表（博客園）](https://www.cnblogs.com/weifeng1463/p/8662241.html)
master 端：
binlog-do-db 二進制日誌記錄的資料庫（多資料庫用逗號，隔開）
binlog-ignore-db 二進制日誌中忽略資料庫（多資料庫用逗號，隔開）

slave 端
replicate-do-db 設定需要複製的資料庫（多資料庫使用逗號，隔開）
replicate-ignore-db 設定需要忽略的複製資料庫（多資料庫使用逗號，隔開）
replicate-do-table 設定需要複製的表
replicate-ignore-table 設定需要忽略的複製表
replicate-wild-do-table 同 replication-do-table 功能一樣，但是可以通配符
replicate-wild-ignore-table 同 replication-ignore-table 功能一樣，但是可以加通配符

原文：[MySQL 主從同步資訊查詢（部落格）](https://blog.51cto.com/yueyinsha/5243070)

# 主從同步故障排除

[MySQL Replication Error 1236（Longwin）](https://blog.longwin.com.tw/2013/09/mysql-replication-error-1236-fix-2013/)
[AWS DMS CDC Error 1236（AWS）](https://aws.amazon.com/cn/premiumsupport/knowledge-center/dms-cdc-error-1236-msql/)

# Foreign Key 命名規則

會注意到這個的原因是因為，資料庫報錯 1022
[MySQL Error 1022（Stack Overflow）](https://stackoverflow.com/questions/15014592/mysql-error-1022-when-creating-table)
[Foreign Key 命名策略（Stack Overflow）](https://stackoverflow.com/questions/199498/foreign-key-naming-scheme)

# 先 JOIN 再查詢，先查詢再 JOIN

其實我們不知道 DB 的行為會是怎樣，所以一切都需要驗證之後才知道
[JOIN vs 子查詢（博客園）](https://blog.csdn.net/qq_15329947/article/details/96482433)
