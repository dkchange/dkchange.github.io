---
title: 不常用會忘的linux指令
date: 2019-11-07 16:17:32
tags:
  - linux
categories:
  - [DevOps路線, 管理伺服器, 作業系統]
---

# 改變資料夾底下檔案的預設權限
使用場景，常常在遠程修改線上的電腦(例如docker container)，要透過網頁走php-fpm新增檔案， 但是資料夾775權限 group stanley一直都不是 nginx 在php-fpm設定的www-data，所以php-fpm無權限新增
如果你是用container，你會發現ll指令會print出userid 1000和groupid 1000
此時你需要在container增加user
> useradd -u 1000 stanley
此時我們可以把www-data加到stanley的group
> usermod -a -G stanley www-data
加完之後確認是否有在群組內
> getent group stanley
帳號重登一次電腦，如果是container就restart
之後www-data就有權限創檔案了
但是www-data創建出來的檔案換成遠端的stanley無法對其進行修改
因為檔案的user和group都是www-data
此時我們在透過網頁創建檔案之前
> chmod g+s .;
This command sets the group ID (setgid) on the current directory, written as ..
This means that all new files and subdirectories created within the current directory inherit the group ID of the directory, rather than the primary group ID of the user who created the file. This will also be passed on to new subdirectories created in the current directory.
g+s affects the file's group ID but does not affect the owner ID.

Note that this applies only to newly-created files. Files that are moved (mv) into the directory are unaffected by the setgid setting. Files that are copied with cp -p are also unaffected.
chmod g+s <directory> //set gid

特殊權限
> setfacl -m u:stanley:rwx /www/index.php
> setfacl -d -m g::rwx /<directory> //set group to rwx default
> setfacl -d -m o::rx /<directory> //set other
setfacl 命令是用来在命令行里设置 ACL（访问控制列表)
> getfacl /<directory>
getfacl 來查看檔案的所有權限，也會把特殊權限的使用者列出來

# 不存在的使用者

[Is it ok to have files owned by a non-existent user?](https://unix.stackexchange.com/questions/305170/is-it-ok-to-have-files-owned-by-a-non-existent-user)
Yes, it's fine.
在掛載 docker volume 的時候就會發現檔案的 user 是 1001 這個 uid

# hitory 儲存指令

有的時候你有沒有想過為什麼 git bash 沒有儲存你之前打的指令，
因為你沒有打 exit 或是 ctrl + c ，你直接就跳離開，這樣他不會儲存
.bash_history 這個檔案

# window 打開檔案縂管

要在 window 打開檔案縂管

> start .

# 环境变数

有的时候我们软体不是安装在全局的资料夹，系统在全局资料夹底下就找不到该软体，那我们可以怎么做？
就是设定环境变数，这样以后下程式指令就不会找不到 command 了，所以
设定取决于你是用那一款 shell 软体
https://stackoverflow.com/questions/25373188/how-to-place-the-composer-vendor-bin-directory-in-your-path

## linux 變數設定

linux
https://www.cnblogs.com/flying-tiger/p/5616934.html
https://blog.csdn.net/GYQJN/article/details/50818231

## windows 變數設定

假如在环境变量新增了
变量名 stanley
变量值 php D:\stanley.phar
那 CMD 呼叫时
%stanley% run
相当于
php D:\stanley.phar run

http://batcheero.blogspot.com/2008/02/set-and-setx.html
https://www.mobile01.com/topicdetail.php?f=300&t=723300

# 安裝 GD Graphics Library

GD Library extension not available with this PHP installation Ubuntu Nginx
https://stackoverflow.com/questions/34009844/gd-library-extension-not-available-with-this-php-installation-ubuntu-nginx

# 找 ip

ifconfig | grep 192

ifconfig: command not found
netstat command not found
這是因為在 RHEL / CentOS 7 開始, 最小化安裝不會包括 ifconfig 及 netstat 等工具, 以前在 CentOS 5 及 6 都是預設安裝的.
yum install net-tools

# 查看網路使用狀態

netstat -natpe

# 列出被程序開啟的檔案

lsof -i
COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME
nginx 9 root 8u IPv4 10172200 0t0 TCP \*:http (LISTEN)
php-fpm 12 root 9u IPv4 10171176 0t0 TCP localhost:cslistener (LISTEN)

# 删除资料夹和檔案

rm -rf letters/

# 查看已安装套件

apt-get 也是 dpkg 的包装
直接使用 dpkg -l 来查看已经安装了的软件

> dpkg -l | grep php

# 查询安装路径

> dpkg -L 软件名
> whereis php7.3

# 切换 PHP 版本

php cli 可以透過 update-alternatives 來進行版本切換
[在 ubuntu 安裝多版本 PHP](https://xenby.com/b/169-%E6%95%99%E5%AD%B8-%E5%9C%A8ubuntu%E5%AE%89%E8%A3%9D%E5%A4%9A%E7%89%88%E6%9C%ACphp-apache)

> sudo update-alternatives --set php /usr/bin/php7.3
> sudo update-alternatives --set php /usr/bin/php5.6

# 查看 linux 版本

> ls -l /etc/\*release
> lsb_release -a
> uname -a
> cat /proc/version

# 查看权限

> ls -la /root

# 關閉 xwindow 的視窗

之後點選對應的視窗 x 按鈕

> xkill

# 看文件最後幾行

tail -n 1000 看最後 1000 行
tail -n +1000 看 1000 行之後的內容

# Linux 如何找出佔用較大空間的檔案

最近家中的 linux 突然磁碟空間爆增!!!

到底是什麼檔案佔用了這些空間?

在找出大檔案時,又怎麼知道 Linux 剩餘磁碟空間呢?

df -h 就可以很清楚知道目前磁碟空間是使用多少了

至於如何找到佔用的檔案,可以利用以下指令

du: 計算目錄所使用的空間
sort: 將輸入的資料排序
head: 將輸入資料的最開頭幾行資料輸出

像如要找到 home 下最大前 5 名如下

du -a /home | sort -n -r | head -n 5

至於要找到磁碟佔用最大的檔案呢?

方式有好幾種:

第一種方式:

像是我們可以先至根目錄下利用

指令 du -h --max-depth=1

–max-depth 是表示查詢子目錄的層級

就可查到目錄佔用的情形,再到較大的目錄,重覆利用此指令去找出佔用較大的檔案

第二種方式:

利用 find 指令如

find / -type f -size +5G

我們可以利用此種方式找出大於 5G 的檔案

第三種方式:

find / -type f -exec du {} \; 2>/dev/null |
sort -n | tail -n 10 | xargs -n 1 du -h 2>/dev/null
"find / -type f" 的意思是「搜尋根目錄中的所有檔案」。

-exec du {} \;" 代表「每個找到的檔案都用 du 指令執行以取得以 bytes 為單位的檔案大小資訊」。

"2>/dev/null" 是指將所有的錯誤訊息丟棄。

"sort -n" 會將所有的檔案依大小列出，

"tail -n 10" 則是顯示最後 10 筆，兩個指令合起來就會顯示出依大小排序的前 10 大檔案

# symlink

ln -s /var/www /home/stanley/sites/www

# cat

將一個文件合併到另一個文件
cat a.sh >> b.py

```console
cat <<EOF >file2
> 111
> 5555
> 333
> EOF
```

# cd 回上一頁

> cd -

# 目前所在終端

tty

# 背景執行

sleep 5000 &
但終端機管掉就會消失
nohup sleep 5000 &
終端機關掉也不會消失

# 查詢進程

ps aux | grep sleep

# 暫時切換前後台

bg
fg
那如果是 vim 編輯到一半可以^+Z 到背景
fg 繼續回來編輯

# | 一個命令的輸出是下一個命令的輸入

## |tee 把內容複製到某文件
會覆蓋掉檔案
例如
date > date.txt
date |tee date.txt

## >> 把內容複製到某文件的最後一行
例如
date >> date.txt

# SSH 複製檔案

scp ubuntu@168.138.40.82:/srv/www/happypanda.subnet.vcn.oraclevcn.com/current/happypanda_subnet_vcn_oraclevcn_com_production-2020-08-30-5a26f48.sql /home/stanley/Downloads

https://discourse.roots.io/t/migration-from-bedrock-to-a-normal-install/12551

# UNPROTECTED PRIVATE KEY FILE

https://stackabuse.com/how-to-fix-warning-unprotected-private-key-file-on-mac-and-linux/

把權限改一下即可
sudo chmod 600 /path/to/my/key.pem

# 匯入 匯出 sql

sudo wp db export --add-drop-table --allow-root

docker-compose run --rm wordpress-cli db import happypanda_subnet_vcn_oraclevcn_com_production-2020-08-30-7da1ec2.sql

# 開機就會執行的指令
[Linux中设置服务自启动的三种方式](https://www.cnblogs.com/nerxious/archive/2013/01/18/2866548.HTML)

會放在 /etc/rc.local 這個檔案裡面

大多時候我們可以在/etc/rc.local 中寫一些命令來啓動自己的程序或服務，但是配置後無法啓動，查看了下是 rc-local.service 未啓動
ConditionFileIsExecutable=/etc/rc.d/rc.local was not met
默認情況下，使用上面的命令無法啓動 rc-local.service 服務，原因是需要兩處文件都設置可執行權限，但是 /etc/rc.d/rc.local 默認沒有可執行權限
https://stackoverflow.com/questions/43671482/how-to-run-docker-compose-up-d-at-system-start-up

## 設置可執行權限

[root@master ~]# chmod +x /etc/rc.d/rc.local
[root@master ~]# chmod +x /etc/rc.local

# 檢查 TCP port 有沒有開

telnet 域名 port 號

# 檢查域名對應 ip

ping 域名

# iptable 規則必須按順序放 然後重啟

https://www.itread01.com/articles/1487665153.html
端口放行條目，請放在下列條目之前，然後修改後，重啟防火墻服務
-A INPUT -j REJECT --reject-with icmp-host-prohibited -A FORWARD -j REJECT --reject-with icmp-host-prohibited

service iptables restart

阿如果不想電腦重開機規則不見的話
可以修改以下檔案
/etc/sysconfig/iptables
或
service iptables save
也會存進去
/etc/sysconfig/iptables

在不然就是
保存规则：#iptables-save >/etc/iptables-script
恢复规则：#iptables-restore>/etc/iptables-script
开机自动恢复规则，把恢复命令添加到启动脚本：echo '/sbin/iptables-restore /etc/iptables-script' >>/etc/rc.d/rc.local

# 查看服務列表

service --status-all
chkconfig --list

systemctl list-units --type=service

# 查看硬碟用量

https://blog.csdn.net/qq_35076663/article/details/103556988?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.add_param_isCf&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.add_param_isCf

du -hsx /data/www/lotteryhub/daemon/\* | sort -hr | head

# 修改使用者群組

當你修改使用者群組時，記得要將使用者登出後再登入，這樣才會成功被納入該群組
要知道有沒有被列入群組之中，用`id`指令即可查看

# 掛載硬碟

mount /dev/sda1 /media/stanley/hdd

# 找出重疊的指令

grep -r listen /etc/nginx/\*

# cron 自己的日誌
https://serverfault.com/questions/136461/how-to-check-cron-logs-in-ubuntu?rq=1

# cron log 分檔
logrotate
https://stackoverflow.com/questions/53366062/creating-cron-job-that-sends-output-to-file-every-day-and-overwrites-this-file-e


# linux 查看环境变量和修改环境变量

“/etc/profile”对系统里所有用户都有效，用户主目录下的“.bash_profile”只对这个用户有效。
https://blog.csdn.net/w6028819321/article/details/21600423

# 搬移檔案，包括隱藏檔

```bash
find . -maxdepth 1 -exec mv {} .. \;
```
# 新增加的群組組員無法在有群組權限的資料夾新增檔案

1. 登出後重登
2. https://superuser.com/questions/1620868/user-added-to-group-cant-create-files-or-folders-inside-group-owned-folder

https://superuser.com/questions/665057/group-member-cannot-create-files-directories-in-folder
