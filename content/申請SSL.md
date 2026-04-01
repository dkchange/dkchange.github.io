---
title: 申請SSL
date: 2019-11-06 18:31:21
tags: https
categories:
  - DevOps路線
  - 網路與安全
  - https
---

# 自簽名憑證

因為開發環境沒有 domain，可有些技術又強制要走 [[HTTPS]]，像是 [[WebRTC]] 的其中一個環節（[帆軟說明](https://help.fanruan.com/finereport/doc-view-2662.html)）

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

在 Chrome 的設定中，匯入憑證，再啟動 Chrome，就大功告成了（[iT 邦教學](https://ithelp.ithome.com.tw/articles/10230157)（連結已失效））

# 憑證取得方法

憑證有分等級，最簡單的就是 **[[DV]] (Domain Validation)** - 僅驗證網域擁有權（傳檔案驗證 OR DNS 驗證）然後就發給你憑證，
那高級一點的就須要人工的身份證明之類的東西（[[OV]]/[[EV]]）
申請 DV 憑證有以下方法：

1. 手動產出 **[[私密金鑰]] (Private Key)**，再用它生成 **[[CSR]] (憑證簽署請求檔)**，拿著 CSR 去跟各大 [[CA]] 申請，驗證網域擁有權後，拿回**憑證**和**[[中繼憑證]]**
2. 最簡單的方法，就是網路上有人做跟各憑證平台串接，做了方法 1 的事，生出 **密鑰、憑證、中繼憑證** 之後我們再載回來用，像是 [SSL For Free](https://free.com.tw/ssl-for-free/)
3. 用自動化工具（優點是可以自動排程更新），DNS 驗證是透過域名商提供的 API 自動添加 [[TXT]] 紀錄來達成驗證，常見工具是 [[acme.sh]]

# [[Let's Encrypt]]

Let’s Encrypt 基於安全考量，沒有開放直接從網站輸入私密金鑰、CSR 檔案來取得憑證的介面，必須使用自動化工具。

Let's Encrypt 免費 [[SSL]] 憑證只有**三個月**的有效期，所以看來只能採第三種方法（使用工具自動更新）。

---

# [[acme.sh]]

acme.sh 是一套實現 [[ACME]] (Automated Certificate Management Environment) 協定的工具，可以用來自動申請和管理 Let's Encrypt 憑證。
官網也有推別款，但我找到的[如何在 Linux 作業系統上免費申請 Let's Encrypt 的 SSL 憑證，並實現自動化申請和套用](https://magiclen.org/simple-ssl-acme-cloudflare/)介紹的是這個

# 安裝 acme.sh

接下來記錄安裝過程，因為有遇到困難，所以才有這篇 XD
跟著[說明書](https://github.com/Neilpang/acme.sh/wiki/%E8%AF%B4%E6%98%8E)做

在做這些動作之前，請切換到 root 身份（不要使用 sudo），因為安裝過程會把 acme.sh 安裝到 root 的目錄，如果用 sudo 可能會裝到其他位置

> [do not run as sudo](https://github.com/Neilpang/acme.sh/issues/2124) - 作者建議不要用 sudo 安裝

```bash
# 切換到 root
sudo su

# 下載安裝
curl https://get.acme.sh | sh

# 更新
acme.sh --upgrade --auto-upgrade
```

出問題了怎麼辦？[acme.sh 官方 issue](https://github.com/Neilpang/acme.sh/issues/2149) 有解決方案。

```bash
source ~/.bashrc
```

```bash
# 申請憑證（HTTP 驗證）
acme.sh --issue --nginx -d yourdomain.com

# 申請憑證（DNS 手動驗證）
acme.sh --issue -d yourdomain.com --dns --yes-I-know-dns-manual-mode-enough-go-ahead-please
```

下次手動更新的時間，我們給他記起來：

```
[Wed Nov  6 11:26:47 UTC 2019] Skip, Next renewal time is: Sun Jan  5 11:05:26 UTC 2020
[Wed Nov  6 11:26:47 UTC 2019] Add '--force' to force to renew.
```

接著下次就這樣：

```bash
acme.sh --renew -d yourdomain.com --yes-I-know-dns-manual-mode-enough-go-ahead-please
```

# Nginx 設定

先把這兩個檔案搬到讀的到的目錄：

```bash
cp fullchain.cer /home/fullchain.cer
cp yourdomain.com.key /home/yourdomain.com.key
```

接著設定 [[Nginx]] 設定檔：

```nginx
server {
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;

    ssl_certificate /home/fullchain.cer;
    ssl_certificate_key /home/yourdomain.com.key;

    server_name yourdomain.com;
}

server {
    listen 80;
    listen [::]:80;

    server_name yourdomain.com;
    return 301 https://$host$request_uri;
}
```

服務重啟：

```bash
sudo systemctl restart nginx
```

AWS 記得開 443 port。

Let’s Encrypt 也支援萬用 [[SSL]] 憑證（[[Wildcard]]），不限個數的子網域，但是[只支援 DNS 驗證](https://github.com/Neilpang/acme.sh/issues/1433#issuecomment-377242752)

補憑證時下面這行因為萬用網域 ([[Wildcard]]) 只能使用 DNS 驗證，HTTP 驗證（`-w` 網頁根目錄）不支援：

```bash
acme.sh --issue -w /var/www/html/public/ -d *.yourdomain.com
```

所以 nginx 的 conf 檔也要改：

```nginx
server_name yourdomain.com *.yourdomain.com;
```

改成寫死的 admin 子網域來驗證。原因是當我們設定了 80 Port 自動轉向 HTTPS 之後，HTTP 驗證會被 301 轉向擋住，無法正確驗證，所以需要指定一個子網域來進行 HTTP 驗證：

```bash
acme.sh --issue -w /var/www/html/public/ -d admin.yourdomain.com
```

nginx：

```nginx
server_name yourdomain.com admin.yourdomain.com;
```

哪天有閒錢再來買 GoDaddy 之類的，像[這篇](https://footmark.info/linux/centos/acmesh-godaddy-letsencrypt-wildcard/)和 [acme.sh 文件](https://github.com/Neilpang/acme.sh/wiki/How-to-issue-a-cert#3-multiple-domains-san-mode--hybrid-mode)

# 相關副檔名

實在很令人混亂的東西，有時後叫 pem，有時後叫 crt，也有 cert 等等，結論就是我把 crt / key / ca 塞入的就叫 pem（[OpenSSL 檔案格式轉換](https://ssorc.tw/7142/openssl-%E6%8C%87%E4%BB%A4-command-line-%E8%BD%89%E6%AA%94-pem-der-p7b-pfx-cer/)）
