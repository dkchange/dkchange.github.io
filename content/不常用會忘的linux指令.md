---
title: 不常用會忘的 Linux 指令
date: 2019-11-07 16:17:32
tags:
  - linux
  - 作業系統
  - 管理伺服器
categories:
  - DevOps路線
  - 日記
---

# 改變資料夾底下檔案的預設權限

使用場景：常常在遠端修改線上的電腦（例如 [[Docker]] container），要透過網頁走 [[PHP-FPM]] 新增檔案。  
但是資料夾雖然是 `775` 權限，group `stanley` 一直都不是 php-fpm 設定的 `www-data`，所以 php-fpm 無權限新增檔案。

如果你是用 container，你會發現 `ll` 指令可能會印出 `userid 1000` 和 `groupid 1000`。  
此時你需要在 container 增加 user：

```bash
useradd -u 1000 stanley
```

此時我們可以把 `www-data` 加到 `stanley` 的 group：

```bash
usermod -a -G stanley www-data
```

加完之後確認是否有在群組內：

```bash
getent group stanley
```

帳號要重新登入一次；如果是 container，通常就 restart。

之後 `www-data` 就有權限創檔案了。  
但是 `www-data` 創建出來的檔案，換成遠端的 `stanley` 可能還是無法修改，因為新檔案的 user 和 group 可能都是 `www-data`。

此時，在透過網頁建立檔案之前，可以先在目錄上設定 setgid：

```bash
chmod g+s .
```

這個指令會替目前目錄加上 group ID（setgid）位元。  
它的作用是：之後在這個目錄底下新建立的檔案與子目錄，會繼承**該目錄的群組**，而不是建立者自己的主要群組。  
這個效果也會傳遞到新建立的子目錄。

注意：

- `g+s` 影響的是 **group**，不影響 owner。
- 這只對**新建立**的檔案有效。
- 用 `mv` 移進來的檔案通常不受影響。
- 用 `cp -p` 保留原權限複製的檔案，也可能不受影響。
- `setgid` 只保證繼承群組，**不保證**新檔案一定有 group write 權限，這還會受 `umask` 影響。

```bash
chmod g+s <directory>   # setgid on directory
```

## 特殊權限 ACL

```bash
setfacl -m u:stanley:rwx /www/index.php
setfacl -d -m g::rwx /<directory>   # 預設 group 權限
setfacl -d -m o::rx /<directory>    # 預設 other 權限
```

`setfacl` 命令是用來在命令列設定 ACL（Access Control List，存取控制列表）。

```bash
getfacl /<directory>
```

`getfacl` 用來查看檔案或目錄的所有權限，也會列出 ACL 特殊權限使用者。

---

# 不存在的使用者

[Is it ok to have files owned by a non-existent user?](https://unix.stackexchange.com/questions/305170/is-it-ok-to-have-files-owned-by-a-non-existent-user)

Yes, it's fine.

在掛載 Docker volume 的時候，就會發現檔案的 user 可能是 `1001` 這種 UID。  
這通常代表系統找不到對應帳號名稱，但檔案仍然是由這個 UID 擁有。

---

# history 儲存指令

有的時候你有沒有想過，為什麼 Git Bash 沒有儲存你之前打的指令？

因為你沒有正常離開 shell，例如沒有打 `exit`，或 shell 被異常中斷、視窗直接關閉，這樣 history 可能不會即時寫入。

history 常見儲存在：

```bash
.bash_history
```

---

# Windows 打開檔案總管

要在 Windows 打開目前資料夾的檔案總管：

```bash
start .
```

---

# 環境變數

有時候我們的軟體不是安裝在全域資料夾，系統在預設路徑底下找不到該軟體，這時就可以設定環境變數。  
設定完成後，以後下指令時就比較不會遇到 `command not found`。

設定方式會取決於你使用哪一款 shell。

[Stack Overflow 討論](https://stackoverflow.com/questions/25373188/how-to-place-the-composer-vendor-bin-directory-in-your-path)

## Linux 變數設定

[[Linux]]  
[部落格教學](https://www.cnblogs.com/flying-tiger/p/5616934.html)  
[CSDN 教學](https://blog.csdn.net/GYQJN/article/details/50818231)

## Windows 變數設定

假如在環境變數新增了：

- 變數名：`stanley`
- 變數值：`php D:\stanley.phar`

那 CMD 呼叫時：

```bat
%stanley% run
```

相當於：

```bat
php D:\stanley.phar run
```

[Batcheero 教學](http://batcheero.blogspot.com/2008/02/set-and-setx.html)  
[Mobile01 討論](https://www.mobile01.com/topicdetail.php?f=300&t=723300)

---

# 安裝 GD Graphics Library

GD Library extension not available with this PHP installation Ubuntu Nginx

[Stack Overflow 討論](https://stackoverflow.com/questions/34009844/gd-library-extension-not-available-with-this-php-installation-ubuntu-nginx)

---

# 找 IP

```bash
ifconfig | grep 192
```

如果出現：

```bash
ifconfig: command not found
netstat: command not found
```

這是因為在 RHEL / CentOS 7 開始，最小化安裝不一定會包含 `ifconfig` 及 `netstat` 等工具。  
以前在 CentOS 5 / 6 常常是預設安裝。

可以安裝：

```bash
yum install net-tools
```

---

# 查看網路使用狀態

```bash
netstat -natpe
```

---

# 列出被程序開啟的檔案

```bash
lsof -i
```

範例：

```text
COMMAND PID USER     FD   TYPE DEVICE   SIZE/OFF NODE NAME
nginx   9   root     8u   IPv4 10172200 0t0      TCP *:http (LISTEN)
php-fpm 12  root     9u   IPv4 10171176 0t0      TCP localhost:cslistener (LISTEN)
```

---

# 刪除資料夾和檔案

```bash
rm -rf letters/
```

---

# 查看已安裝套件

`apt-get` 也是 `dpkg` 的包裝工具。  
直接使用 `dpkg -l` 也可以查看已經安裝的軟體。

```bash
dpkg -l | grep php
```

---

# 查詢安裝路徑

```bash
dpkg -L 軟體名
whereis php7.3
```

---

# 切換 PHP 版本

php CLI 可以透過 `update-alternatives` 來進行版本切換。

[在 ubuntu 安裝多版本 PHP](https://xenby.com/b/169-%E6%95%99%E5%AD%B8-%E5%9C%A8ubuntu%E5%AE%89%E8%A3%9D%E5%A4%9A%E7%89%88%E6%9C%ACphp-apache)

```bash
sudo update-alternatives --set php /usr/bin/php7.3
sudo update-alternatives --set php /usr/bin/php5.6
```

---

# 查看 Linux 版本

```bash
ls -l /etc/*release
lsb_release -a
uname -a
cat /proc/version
```

---

# 查看權限

```bash
ls -la /root
```

---

# 關閉 X Window 的視窗

先執行：

```bash
xkill
```

之後點選對應的視窗 X 按鈕，或直接點選要關閉的視窗即可。

---

# 看文件最後幾行

```bash
tail -n 1000
```

看最後 1000 行。

```bash
tail -n +1000
```

看第 1000 行之後的內容。

實際使用通常會加上檔名，例如：

```bash
tail -n 1000 app.log
tail -n +1000 app.log
```

---

# Linux 如何找出佔用較大空間的檔案

最近家中的 Linux 突然磁碟空間爆增。  
到底是什麼檔案佔用了這些空間？

先用下面指令看整體磁碟空間：

```bash
df -h
```

至於如何找到佔用空間大的檔案，可以利用以下指令：

- `du`：計算目錄所使用的空間
- `sort`：將輸入資料排序
- `head`：將輸入資料最前面的幾行輸出

像是要找出 `/home` 下最大的前 5 名：

```bash
du -a /home | sort -n -r | head -n 5
```

至於要找到磁碟佔用最大的檔案，方式有幾種。

## 第一種方式

先到根目錄下利用：

```bash
du -h --max-depth=1
```

`--max-depth` 是表示查詢子目錄的層級。

可以先找出哪個目錄最大，再進入較大的目錄重複使用這個指令。

## 第二種方式

利用 `find` 指令：

```bash
find / -type f -size +5G
```

可以找出大於 5GB 的檔案。

## 第三種方式

```bash
find / -type f -exec du {} \; 2>/dev/null | sort -n | tail -n 10 | xargs -n 1 du -h 2>/dev/null
```

說明：

- `find / -type f`：搜尋根目錄中的所有檔案
- `-exec du {} \;`：對每個找到的檔案執行 `du`
- `2>/dev/null`：把錯誤訊息丟棄
- `sort -n`：依大小排序
- `tail -n 10`：顯示最後 10 筆，也就是最大的 10 筆

---

# symlink

```bash
ln -s /var/www /home/stanley/sites/www
```

---

# cat

將一個文件合併到另一個文件：

```bash
cat a.sh >> b.py
```

建立檔案：

```console
cat <<EOF > file2
111
5555
333
EOF
```

---

# cd 回上一頁

```bash
cd -
```

---

# 目前所在終端

```bash
tty
```

---

# 背景執行

```bash
sleep 5000 &
```

但終端機關掉就會消失。

```bash
nohup sleep 5000 &
```

終端機關掉也不會消失。

---

# 查詢進程

```bash
ps aux | grep sleep
```

---

# 暫時切換前後台

```bash
bg
fg
```

如果是 `vim` 編輯到一半，可以按 `Ctrl + Z` 丟到背景，之後再用 `fg` 回來。

---

# `|` 一個命令的輸出是下一個命令的輸入

## `tee` 把內容複製到某文件

會覆蓋掉檔案，例如：

```bash
date > date.txt
date | tee date.txt
```

## `>>` 把內容加到某文件最後一行

例如：

```bash
date >> date.txt
```

---

# SSH 複製檔案

```bash
scp ubuntu@168.138.40.82:/srv/www/happypanda.subnet.vcn.oraclevcn.com/current/happypanda_subnet_vcn_oraclevcn_com_production-2020-08-30-5a26f48.sql /home/stanley/Downloads
```

[Roots.io 討論](https://discourse.roots.io/t/migration-from-bedrock-to-a-normal-install/12551)

---

# UNPROTECTED PRIVATE KEY FILE

[StackAbuse 教學](https://stackabuse.com/how-to-fix-warning-unprotected-private-key-file-on-mac-and-linux/)

把權限改一下即可：

```bash
sudo chmod 600 /path/to/my/key.pem
```

---

# 匯入 / 匯出 SQL

```bash
sudo wp db export --add-drop-table --allow-root
```

```bash
docker-compose run --rm wordpress-cli db import happypanda_subnet_vcn_oraclevcn_com_production-2020-08-30-7da1ec2.sql
```

---

# 開機就會執行的指令

[博客園教學](https://www.cnblogs.com/nerxious/archive/2013/01/18/2866548.HTML)

有些系統會放在 `/etc/rc.local` 這個檔案裡。

大多時候我們可以在 `/etc/rc.local` 中寫一些命令來啟動自己的程序或服務。  
但配置後如果無法啟動，可能是 `rc-local.service` 沒啟用，或 `rc.local` 沒有執行權限。

例如曾看過：

```text
ConditionFileIsExecutable=/etc/rc.d/rc.local was not met
```

預設情況下，可能需要把下列檔案都設為可執行：

[Stack Overflow 討論](https://stackoverflow.com/questions/43671482/how-to-run-docker-compose-up-d-at-system-start-up)

## 設置可執行權限

```bash
chmod +x /etc/rc.d/rc.local
chmod +x /etc/rc.local
```

---

# 檢查 TCP port 有沒有開

```bash
telnet 域名 port號
```

---

# 檢查域名對應 IP

```bash
ping 域名
```

---

# iptables 規則必須按順序放，然後重啟

[IT .read01 教學](https://www.itread01.com/articles/1487665153.html)

端口放行條目，請放在下列條目之前，然後修改後重啟防火牆服務：

```text
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
```

```bash
service iptables restart
```

如果不想電腦重開機規則消失，可以修改以下檔案：

```bash
/etc/sysconfig/iptables
```

或使用：

```bash
service iptables save
```

也會存進：

```bash
/etc/sysconfig/iptables
```

另外也可以：

```bash
iptables-save > /etc/iptables-script
iptables-restore < /etc/iptables-script
```

開機自動恢復規則，可把恢復命令加到啟動腳本：

```bash
echo '/sbin/iptables-restore /etc/iptables-script' >> /etc/rc.d/rc.local
```

---

# 查看服務列表

```bash
service --status-all
chkconfig --list
systemctl list-units --type=service
```

---

# 查看硬碟用量

[CSDN 教學](https://blog.csdn.net/qq_35076663/article/details/103556988)

```bash
du -hsx /data/www/lotteryhub/daemon/* | sort -hr | head
```

---

# 修改使用者群組

當你修改使用者群組時，記得要將使用者登出後再登入，這樣才會成功被納入該群組。  
要知道有沒有被列入群組之中，用 `id` 指令即可查看。

---

# 掛載硬碟

```bash
mount /dev/sda1 /media/stanley/hdd
```

---

# 找出重疊的指令 / 設定

```bash
grep -r listen /etc/nginx/*
```

---

# cron 自己的日誌

[https://serverfault.com/questions/136461/how-to-check-cron-logs-in-ubuntu?rq=1](https://serverfault.com/questions/136461/how-to-check-cron-logs-in-ubuntu?rq=1)

---

# cron log 分檔

`logrotate`

[https://stackoverflow.com/questions/53366062/creating-cron-job-that-sends-output-to-file-every-day-and-overwrites-this-file-e](https://stackoverflow.com/questions/53366062/creating-cron-job-that-sends-output-to-file-every-day-and-overwrites-this-file-e)

---

# Linux 查看環境變數和修改環境變數

`/etc/profile` 對系統裡所有使用者都有效，使用者主目錄下的 `.bash_profile` 只對這個使用者有效。

[CSDN 教學](https://blog.csdn.net/w6028819321/article/details/21600423)

---

# 搬移檔案，包括隱藏檔

```bash
find . -maxdepth 1 -exec mv {} .. \;
```

注意：這種寫法很危險，可能會連 `.` 本身或不想移動的項目也一起處理。  
使用前建議先用 `find . -maxdepth 1` 確認內容。

---

# 新增加的群組組員無法在有群組權限的資料夾新增檔案

1. 登出後重登
2. 檢查目錄是否真的有 group write / execute 權限
3. 檢查是否有 ACL 覆蓋原本權限

- [https://superuser.com/questions/1620868/user-added-to-group-cant-create-files-or-folders-inside-group-owned-folder](https://superuser.com/questions/1620868/user-added-to-group-cant-create-files-or-folders-inside-group-owned-folder)
- [https://superuser.com/questions/665057/group-member-cannot-create-files-directories-in-folder](https://superuser.com/questions/665057/group-member-cannot-create-files-directories-in-folder)
