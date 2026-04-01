---
title: 使用docker
date: 2020-04-06 09:32:25
tags:
categories:
  - DevOps路線
  - CI/CD工具
  - Docker
---

# 前言

容器造就了許多現代化服務，諸如 serverless function as a service，然而技術的演變越來越快，那我們就必須跟著學新工具、新流程、新術語

為什麼你需要 Docker，因為你追求效率、速度，在容器化出現之前，你大概 80％ 的時間都在做維運的事，更新、備份、部署
在 Docker 出現之後，你可以更專注於開發

Docker 和 Vagrant 有何不同：

- [Vagrant 與 Docker 定位的反思](http://blog.iwonder.tw/2017/12/27/reflection-of-vagrant-and-docker-positioning/)
- [Docker Data Volume Container 教學](https://peihsinsu.gitbooks.io/docker-note-book/content/docker_data_volume_container.html)

因為工作需要，開始使用 Docker
參考文章：[iThelp Docker 教學](https://ithelp.ithome.com.tw/users/20103456/ironman/1320)
Docker image 是眾多 images 的集合，方便管理

[Docker 學習筆記](https://medium.com/@charliecc/docker-%E7%B7%B4%E7%BF%92%E7%AD%86%E8%A8%98-vol-1-df59d4fdc9c9)

第二是 Docker container 也跟寫程式一樣，有 decouple（解耦）的概念，我的開發環境通常會是 PHP + Nginx，剛開始建的時候曾為了一個問題糾結很久：我到底應該要在

- PHP 的 image 上裝 Nginx？
- Nginx 的 image 上裝 PHP？
- 用一個乾淨的 Alpine image，在上面裝 PHP + Nginx？

每個都會遇到不同的問題，後來看了一篇分享豁然開朗，兩個 image 分開，各司其職，這樣容易維護，調整功能也非常容易。
有時候因為正式部署環境或其他需求的關係，你可能需要建成單一 image 來作業，這邊以開發時期，快速調整為前提

# Alpine Linux 挑戰最小 docker image OS

官方都會出一個服務例如 php，nginx，mysql
然而這些服務，你連接到 php container 的時候，bash 下 mysql 在該 container 並找不到 mysql
所以各項服務都是由 docker 來橋接的

# exited

有的時候官方的映像檔啟動時並不如你預期
可能會出現 status exited 1
這時候就來查 log 啦
docker-compose logs 服務名
docker logs 容器 id

## 如何開發與維運

https://ithelp.ithome.com.tw/articles/10185892

# Docker 目錄解說

[Docker 目錄解說（簡書）](https://www.jianshu.com/p/02c1051245d2)

# 自建環境

sudo docker build . -t myyii --no-cache
sudo
sudo docker run -d --name yii-adv -p 8088:80 \
 -v /home/stanley/Desktop/mydockertest/www/yii-adv:/var/www/yii-adv \
 -v /home/stanley/Desktop/mydockertest/nginx-yii-adv.conf:/etc/nginx/conf.d/default.conf \
 --network=desktop_backend \
 myyii

sudo docker run -d --name yii-basic -p 8089:80 \
 -v /home/stanley/Desktop/mydockertest/www/yii-basic:/var/www/yii-basic \
 -v /home/stanley/Desktop/mydockertest/nginx-yii-basic.conf:/etc/nginx/conf.d/default.conf \
 --network=desktop_backend \
 myyii

[解決 Fully Qualified Domain Name 問題（Stack Overflow）](https://stackoverflow.com/questions/46266527/could-not-reliably-determine-the-servers-fully-qualified-domain-name-how-to)

有成功，但還要在設定 Apache 那些
sudo docker run -d --name mypay --hostname mypay.com -p 8087:80 \
 -v /home/stanley/Desktop/Docker/php5217/weifu:/var/www mypay

docker run mypay 不在背景執行看報什麼錯
docker run -dit mypay 可以強行執行
然後再進去看哪裡錯

[為何 Docker 容器會立即退出（Stack Overflow）](https://stackoverflow.com/questions/28212380/why-docker-container-exits-immediately)

改換
[docker-php-5.2.17-fpm（GitHub）](https://github.com/fifilyu/docker-php-5.2.17-fpm)
docker run -d --name mypay -p 8087:8080 docker-php-5.2.17-fpm
sudo docker run -d --name mypay -v /home/stanley/Downloads/weifu/zfb:/data/web \
-p 8087:8080 \
docker-php-5.2.17-fpm

sudo docker network ls

default.conf /etc/nginx/conf.d/default.conf
sudo docker run -d --name yiitest -p 8088:80 -v /home/stanley/Desktop/mydockertest/www:/var/www --network=desktop_backend myyii

sudo docker exec -it yiitest /bin/bash

sudo stats 會秀出 container 佔用的資源

sudo logs containerName 會 show 出如果執行在前景的 log

# Dockerfile

[Docker CentOS Nginx PHP（GitHub）](https://github.com/kaushalkishorejaiswal/Docker-Centos-Nginx-PHP)

    vi Dockerfile
    撰寫 dockerfile

    ADD nginx.repo /etc/yum.repos.d/nginx.repo
    ADD 把 Local 的檔案複製到 Image 裡

    RUN yum -y install nginx
    RUN Linux 指令
    RUN yum update -y && \
        yum install -y epel-release

**&& \\** 用來連接字串

以下是 CentOS 推薦庫
[CentOS 7 安裝 Repository](https://zoneless.blog/2018/01/07/install-repository-on-centos7/)

    VOLUME ./www
    掛載 local 位置給 container，然而你無法指定 container 的對應的位置
    只能透過指令來完成 A Dockerfile defines how an image is built, not how it's used
    [在 Dockerfile 中新增 Bind Mount（Stack Overflow）](https://stackoverflow.com/questions/47942016/add-bind-mount-to-dockerfile-just-like-volume)
    using -v /path/on/host:/path/in/container to access directories from the host.

    EXPOSE 80
    對外開放 80 port

    WORKDIR /var/www/
    預設工作目錄

    CMD ["supervisord", "-n"]
    在執行 docker run
    之後會直接開啟這個程式

# docker image

    docker build -t imageName . --no-cache

在剛剛的 dockerfile 目錄下指令，客製 docker image

docker run -p 8080:8080 imageName

執行該 image
run 自家專案時，出現了以下錯誤
exited: php5-fpm-log (exit status 1; not expected)
是因為我們有執行 supervisor，所以在前景出現這個正常

所以記得加上 -d 在背景執行
docker run -p 8088:80 -d imageName
加上 port 號的對應之後，就可以用 localhost:8088 進入改該專案

# Bash

    docker run -it --entrypoint bash imageName
    docker exec -it docker_id /bin/bash
    sudo docker exec -it yiitest /bin/bash

進入該 container

當進入容器時，要進行服務開啟會保下列的錯
Failed to get D-Bus connection: Operation not permitted
原因是因為 **_Docker 的設計理念是在容器里面不运行后台服务_**
[Docker 無法使用 systemd（部落格）](https://blog.51cto.com/lizhenliang/1975466)

所以最好一開始就在 dockerfile 寫好要啟動的服務

# Docker Compose

Docker Compose 可使用的指令比 docker 多很多，因為連結各個 container
Docker Compose 雖然是獨立的軟體，但是和 docker 還是有相依性，所以安裝時要注意版本
docker-compose up -d
驗證 docker-compose.yml 內容是否正確
啟動很多個 Docker Container 時，就需要輸入很多次 docker run 指令，另外 container 和 container 之間要做關聯的話也要記得它們之間要如何的連結(link) Container

    volumes:
      - "./m88web:/var/www"
      - "./config/freetds.conf:/etc/freetds.conf"
      - "./config/nginx/backend.conf:/etc/nginx/conf.d/default.conf"

用上面的設定做檔案的替換

    在有 docker-compose.yml 的資料夾底下，下
    sudo docker-compose up -d
    如果檔案有做修改，會把新增的 service 建起來，但已經打開的不會因為你註解就關掉，以防你註解錯直接關掉，引發災情，所以要關掉就手動關吧


    docker info
    列出 docker 相關訊息 image path 等


    sudo docker ps -a
    列出所有運作的 container

    sudo docker-compose stop container_id
    停止單一 container

    sudo docker stop $(sudo docker ps -a -q)
    停止所有 container

    sudo docker rm $(sudo docker rm -a -q)
    移除所有 container

    image has dependent child images
    查出有相依的 images
    sudo docker image inspect  --format='{{.RepoTags}} {{.Id}} {{.Parent}}' $(sudo docker image ls -q --filter since=container_id)

    sudo docker-compose exec serviceName bash
    進入 container 環境，這裡和 docker 不一樣，用的是 yml 檔裡面設定的 serviceName
    而且切記一定要在 docker-compose.yml 的目錄下執行

    exit
    離開 container

    因為 adminer 有版本相關問題，所以決定 rebuild image
    Postgres: ERROR: column d.adsrc does not exist 4.7.5 版本以上才 ok
    sudo docker-compose up -d --no-deps --build adminer
    sudo docker-compose up -d --no-deps --build m88-mapi
    sudo docker-compose build m1-frontend
    不需要去 pull 新版的 adminer image

## "bash": executable file not found in $PATH"

有的精簡的 docker 鏡像使用的 Alpine 這個 Linux 發行版，沒有內建 bash 的。所以應該用 /bin/sh
docker exec -it XXXXX /bin/sh

## 電腦重啟後

電腦重啟之後有設定 restart:always 的服務基本上都會重啟，沒有設定的就要自己手動下指令
sudo docker restart container_name
sudo docker-compose restart service_name

# Container 內指定登入的使用者

會指定使用者權限通常都是有掛載 Host volume 進入容器內，但是您又沒有 root 權限，如果沒有這樣做，
這樣產生出來的檔案都會是 root 權限，一般使用者無法寫入，只能讀取，這時就需要用到此方法。
[Docker 使用者與群組設定（吳咖部落格）](https://blog.wu-boy.com/2019/10/three-ways-to-setup-docker-user-and-group/)
假設我是 stanley 登入 local 端，在 container 裡面是 uid 1001，但沒有此 user，我們可以改用 uid 1001 去 run container，
uid 在 host 和 container 的關係
[UID/GID 對應（Gist）](https://gist.github.com/mrtns/69a92fb387c20bf9add622c4304876b6)
但是這樣又會遇到常駐程式 daemon 無法 run 的問題，
所以我折衷的方法是，用 local root 登入 vsc 做編輯，這樣一樣是 root，編輯檔案就不會有權限問題了，但是這樣就變成 local 端在下指令時都要是 sudo，其也相當不方便
要不然就是要設定資料夾權限全部 777，不懂數字轉換可以到以下網址
[Chmod 計算器](https://chmod-calculator.com/)

## 是否會影響其他的使用者執行

會，git commit 會連權限都 commit 進去，但是因為你的檔案擁有者會隨著 git pull 下來而改變，所以一般都不會受影響
git checkout/diff 略過 file permission 屬性
git config core.fileMode false

[Chmod 權限設定（Stack Overflow）](https://stackoverflow.com/questions/40978921/how-to-add-chmod-permissions-to-file-in-git/40979016)

# 看 Log

docker inspect $container_id
sudo docker logs desktop_m88-mapi_1

# 網路設定

[Docker 網路橋接（Godleon 部落格）](https://godleon.github.io/blog/Docker/docker-network-bridge/)

那通常 docker-compose 之後，你可以在 host 檔上加一些域名，會讓自己方便一些，例如
127.0.0.1 postgres
127.0.0.1 mssql
127.0.0.1 mysql
不全部都是 localhost 看起來比較不會那麼亂，用 adminer 連專案，還會幫你選預設 port 號，很方便喔

# 設定 User

[理解 Docker 中的 UID 和 GID（Medium）](https://medium.com/@mccode/understanding-how-uid-and-gid-work-in-docker-containers-c37a01d01cf)

# Clone a Container

docker commit
阿如果你有掛載東西，掛載的資料不會被複製喔
[Clone Docker Container（Gist）](https://gist.github.com/thaJeztah/8d0e901bd21329d80cf2)

# 利用 SSH 對多台遠端主機下指令

[利用 SSH 控制多台遠端主機（Medium）](https://medium.com/@charliecc/%E5%88%A9%E7%94%A8-ssh-%E5%B0%8D%E5%A4%9A%E5%8F%B0%E9%81%A0%E7%AB%AF%E4%B8%BB%E6%A9%9F%E4%B8%8B%E6%8C%87%E4%BB%A4-a1504cfd1e64)

# 當你用 Docker Compose

'mysql:host=1270.0.1;dbname=yiivue;',
一直失敗
[Docker MySQL 連線被拒（Stack Overflow）](https://stackoverflow.com/questions/40561433/docker-mysql-2002-connection-refused)
要用 service name
'mysql:host=db;dbname=yiivue;',

# 熱門的專案

當你使用的技術比較熱門時，你的資源總是會比較多，可以加快你的學習速度
[Yii2 Starter Kit（GitHub）](https://github.com/yii2-starter-kit/yii2-starter-kit)

# 使用 Alpine NodeJS 時出現 Python 找不到

[NodeJS Docker Image 問題（GitHub Issue）](https://github.com/nodejs/docker-node/issues/384)
這時候可能就要撰寫 dockerfile 了

# Could not reliably determine the server's fully qualified domain name

[Fully Qualified Domain Name 問題（Stack Overflow）](https://stackoverflow.com/questions/46266527/could-not-reliably-determine-the-servers-fully-qualified-domain-name-how-to)

# 使用 Docker Compose 安裝的工具指令

像是我把 node 命名為 webpacker

docker-compose run webpacker npm install --save vue-circle-slider

# 服務

使用 docker-compose 在使用服務的時候，使用 adminer 存取資料庫相關服務，用 localhost:port 號，會有無法存取資料庫的問題，但是在 host 加完服務名稱 mysql，使用 mysql 或 mysql:3306 就可以正常存取，所以在此推測服務也是有對應域名之類的關係，但是自己架的純網站的服務用 用服務名或 localhost:port 號存取 又都沒什麼事，可以正常存取，之後再來探討，應該是官方的 image 有做設定吧？那內部的服務互相溝通舊部需要再填 port 號了，port 號是給給本機外連的程式用的

# 切換使用者之後鍵盤上下鍵失效

after su in container the arrow keys seems not woking
[Notebook Issue（GitHub）](https://github.com/jupyter/notebook/issues/2457)

the issue by explicitly setting the SHELL env var to /bin/bash

所以就 執行一次 /bin/bash 就可以使用 bash 了

# 如何不重啟容器就讓 PHP-FPM 服務 reload config

PHP-FPM is a process manager which supports the USER2 signal, which is used to reload the config file.

From inside the container:

    kill -USR2 1

Outside:

    docker exec -it <mycontainer> kill -USR2 1

Complete example:

    docker run -d --name test123 php:7.1-fpm-alpine
    docker exec -it test123 ps aux
    docker exec -it test123 kill -USR2 1
    docker exec -it test123 ps aux

[重啟 PHP-FPM（Stack Overflow）](https://stackoverflow.com/questions/37806188/how-to-restart-php-fpm-inside-a-docker-container)

# Docker Compose v2

docker-compose 變成插件的形式，所以我們可以加別名解決這個問題
[如何安裝 docker-compose（GitHub Issue）](https://github.com/docker/compose/issues/8575#issuecomment-906298809)

```console
alias docker-compose="docker compose"
```

# 如何清理 Docker Log

```console
truncate -s 0 /var/lib/docker/containers/*/*-json.log
```

[清理 Docker Log（Stack Overflow）](https://stackoverflow.com/questions/42510002/docker-how-to-clear-the-logs-properly-for-a-docker-container)

# /dev/stdout

```console
# 輸出到螢幕上
stanley@localhost:~$ echo '123' > /dev/stdout
```

# 如何清理docker log

```console
truncate -s 0 /var/lib/docker/containers/*/*-json.log
```

https://stackoverflow.com/questions/42510002/docker-how-to-clear-the-logs-properly-for-a-docker-container

# /dev/stdout

```console
# 輸出到螢幕上
stanley@localhost:~$ echo '123' > /dev/stdout
```
