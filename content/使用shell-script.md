---
title: 使用shell script
date: 2020-07-28 09:48:13
tags:
categories:
---

# 前言

[[Shell Script]] 是初階維運必備技能
Shell 是殼的意思，文字介面底下讓我們與系統溝通的一個工具介面，有自己的語法，其速度並沒有傳統程式語言的快
[鳥哥的 Linux 私房菜 - 基礎學習篇](http://linux.vbird.org/linux_basic/0320bash.php)
例如 sh, [[Bash]], csh, tcsh, [[PowerShell]], MSDOS
可以與系統程式語言都可以寫 script，只不過它不附帶工具介面，就不叫它 Shell Script 了
例如 [[Perl]]，[[PHP]]，[[Java]], [[Python]]
為什麼沒有 C 呢？因為 C 不是直譯而是編譯的，
那 Java 呢？因為 Java 編譯出的是位元組碼（需要透過 JDK 來執行），不是機器碼，CPU 不能直接用

那為什麼有了 Python 這些，還需要有 Shell Script 呢？
[Stack Overflow 討論](https://stackoverflow.com/questions/2785155/php-vs-bash-for-cli-scripting/2785163#2785163)

那在這篇探討最通用的 sh
，本身語法難度相較於其他程式語言並不難，難的是你必須要會操作軟體的指令，例如 [[Nginx]] 指令，[[Git]] 指令等等

既然是程式語言，就可以做任何事，但因為速度不快，比較常應用在維運方面，常見的應用

- 初始化（軟體安裝）
- 自動部署（LAMP，LNMP）
- 寫管理應用程式（叢集管理，KVM，[[Nginx]]，[[MySQL]]，批量遠端 RAID）
- Log 分析（grep）
- 自動化備份
- 監控（CPU，Disk）
- 自動擴充（監控 >> API AWS/EC2 >> Shell Script 自動部署）

那以上應用也有不同軟體可以達成，不一定要用 sh，例如老牌 [[Zabbix]] 監控軟體，絕對比自己寫 sh 好

# 常用指令

列出所有 shell
cat /etc/shells

查詢相關配置文件
rpm -qc bash

調適腳本 debug
bash -vx pinggoogle.sh

## alias

查看別名
alias
定義臨時別名
alias stanley = 'date'
定義永久別名
vim .bashrc
取消別名
unalias 別名

## 歷史相關指令

!! 上個命令
!$ 上個命令的最後一個參數

# 規則

# 分類

- login shell（ su - stanley）
  - /etc/profile
  - /etc/bashrc
  - ~/.bash_profile
  - ~/.bashrc
- nologin shell ( su stanley)
  - /etc/bashrc
  - ~/.bashrc

## 檔案開頭

#!/usr/bin/bash
#!/usr/bin/python
指定解釋器

> chmod +x pinggoogle.sh

就可以直接執行
sub shell 會另開 shell

> ./pinggoogle.sh

或是

> . pinggoogle.sh
> source pinggoogle.sh

會影響當前 shell 的執行（目前的視窗）
否則就必須前面加解釋器

sub shell 會另開 shell

> bash pinggoogle.sh

## 兩段不同語言混雜

```sh
#!/usr/bin/bash
echo "im bash"

/usr/bin/python <<-EOF
print "im python"
EOF
echo 'im bash'
```

上面這段代碼，EOF 是約定俗成，可自由定義，其中 <<- 的 - 可以避免縮進問題
例如

```sh
#!/usr/bin/bash
echo "im bash"

/usr/bin/python <<EOF
print "im python"
EOF
```

上面這段去掉 - 就會報錯

## 範例 1

ping 一次

> ping -c1 google.com &>/dev/null && echo "google is alive" || echo "google is down"
> && 和 || 相當於三元運算子
> &>/dev/null 將過程輸出到 /dev/null 不顯示

## Shell 環境變量

相關內容鳥哥的網站都有說
[Linux System 教學](https://www.cntofu.com/book/46/linux_system/1517.md)

set：顯示（設置）shell 變量，包括的私有變量以及用戶變量，不同類的 shell 有不同的私有變量 bash, ksh, csh 每種 shell 私有變量都不一樣

env：顯示（設置）用戶變量

export：顯示（設置）當前導出成用戶變量的 shell 變量。
