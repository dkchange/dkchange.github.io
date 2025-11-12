---
title: 申請SSL
date: 2019-11-06 18:31:21
tags: https
categories:
  - [DevOps路線, 網路與安全, https]
---

# 自簽名

因為開發環境沒有 domain，可有些技術又強制要走 ssl，像是 webrtc 的其中一個環節
https://help.fanruan.com/finereport/doc-view-2662.html

```console
yum install openssl openssl-devel
openssl version -a
openssl genrsa -des3 -out server.key 2048
openssl rsa -in server.key -out server.key
openssl req -new -key server.key -out server.csr
openssl req -new -x509 -key server.key -out ca.crt -days 3650
openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey server.key -CAcreateserial -out server.crt
```

之後在 nginx 做相關的設定

在 Chrome 的設定中，匯入憑證，再啟動 Chrom，就大功告成了
https://ithelp.ithome.com.tw/articles/10230157

# 憑證取得方法

憑證有分等級，最簡單的就是**驗證網域 484 你的**(傳檔案驗證 OR DNS 驗證)然後就發給你憑證，
阿高級一點的就須要人工的身份證明之類的東西
我們申請簡單的憑證，幾個方法

1. 手動生出 **密鑰** 密鑰再用來生成**憑證簽屬請求檔**，再拿這請求檔去跟各大平台，驗證網域 484 你的之後，拿**憑證**和**中繼憑證**回來
2. 最簡單的方法，就是網路上有人做跟各憑證平台串接，做了方法 1 的事，生出 **密鑰、憑證、中繼憑證** 之後我們再載回來用，像是https://free.com.tw/ssl-for-free/
3. 用像是集成方法 2 功能的軟體，缺點是要打指令，優點是他可以自動排程更新憑證，DNS 驗證網域使用域名解析商提供的 API 來添加 TXT 達成驗證，像是 acme.sh

# Let's Encrypt

Let’s Encrypt ，沒有開放直接從網站輸入私密金鑰、CSR 檔案來取得憑證的介面
Let's Encrypt 免費 SSL 憑證只有三個月的有效期
所以看來只能採第三種方法

# acme.sh

ACME(Automated Certificate Management Environment)自動證書管理環境
官網也有推別款，但我找到的[如何在 Linux 作業系統上免費申請 Let's Encrypt 的 SSL 憑證，並實現自動化申請和套用](https://magiclen.org/simple-ssl-acme-cloudflare/)介紹的是這個

# 安裝 acme.sh

接下來記錄安裝過程，因為有遇到困難，所以才有這篇 XD
跟著[說明書](https://github.com/Neilpang/acme.sh/wiki/%E8%AF%B4%E6%98%8E)做

在做這些動作之前，請切到 root，不然會有權限問題，也不要加 sudo 直接做，路徑會不在 root 底下，而且作者這樣說

> [do not run as sudo](https://github.com/Neilpang/acme.sh/issues/2124)

    sudo su
    下載
    curl https://get.acme.sh | sh
    更新
    acme.sh --upgrade --auto-upgrade
    出現問題**acme.sh: command not found**
    解決辦法:https://github.com/Neilpang/acme.sh/issues/2149
    source  ~/.bashrc

    acme.sh --issue --nginx -d dkchange.nctu.me

    手動更新，因為我的DNS是小家nctu.me沒人作API
    acme.sh --issue -d dkchange.nctu.me --dns --yes-I-know-dns-manual-mode-enough-go-ahead-please
    然後憑證就出來了


    下次手動更新的時間，我們給他記起乃
    [Wed Nov  6 11:26:47 UTC 2019] Skip, Next renewal time is: Sun Jan  5 11:05:26 UTC 2020
    [Wed Nov  6 11:26:47 UTC 2019] Add '--force' to force to renew.
    接著下次就醬
    acme.sh --renew -d dkchange.nctu.me  --yes-I-know-dns-manual-mode-enough-go-ahead-please

# nginx 設定

先把這兩個檔案搬到讀的到的目錄

    cp fullchain.cer /home/fullchain.cer
    cp dkchange.nctu.me.key /home/dkchange.nctu.me.key

接著設定 nginx 設定檔

    server {
        listen 443 ssl default_server;
        listen [::]:443 ssl default_server;
        ssl_certificate /home/fullchain.cer;
        ssl_certificate_key /home/dkchange.nctu.me.key;
    }

    server {
            listen 80;
            listen [::]:80;

            server_name dkchange.nctu.me;
            return  301 https://$host$request_uri;
    }

服務重啟

    sudo systemctl restart nginx

AWS 記得開 443port

Let’s Encrypt 也支援萬用 SSL 憑證，不限個數的子網域 但是[只支援 dns 驗證](https://github.com/Neilpang/acme.sh/issues/1433#issuecomment-377242752)

    補憑證時下面這行因為通配符的子網域驗證只適用於-dns 伺服器-nginx 和檔案位置-w都不能用
    acme.sh --issue -w /var/www/html/lighthouse-tutorial/public/  -d *.dkchange.nctu.me

    所以nginx的conf檔也要改
    server_name dkchange.nctu.me *.dkchange.nctu.me;
    改成寫死的admin 驗證方式也要改，因為有設定301重導和https了，所以走模擬server請求的nginx結果會不同
    acme.sh --issue -w /var/www/html/lighthouse-tutorial/public/  -d admin.dkchange.nctu.me
    server_name dkchange.nctu.me admin.dkchange.nctu.me;

哪天有閒錢再來買 godaddy 之類的，像[這篇](https://footmark.info/linux/centos/acmesh-godaddy-letsencrypt-wildcard/)和https://github.com/Neilpang/acme.sh/wiki/How-to-issue-a-cert#3-multiple-domains-san-mode--hybrid-mode

# 相關副檔名

實在很令人混亂的東西，有時後叫 pem，有時後叫 crt，也有 cert 等等，結論就是我把 crt / key / ca 塞入的就叫 pem

https://ssorc.tw/7142/openssl-%E6%8C%87%E4%BB%A4-command-line-%E8%BD%89%E6%AA%94-pem-der-p7b-pfx-cer/
