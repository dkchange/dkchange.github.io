---
title: 使用ansible
date: 2020-09-10 10:39:44
tags:
categories:
---

# 前言

[[Ansible]] 算是一個把 [[YAML]] 檔用到淋漓盡致的軟體，主要運用於同時多台自動化部署，[[Playbook]] 可以說是將 [[YAML]] 檔再做編譯的自創語言

假設我現在有三個節點，controller, node1, node2 我們的 ansible 會安裝在 controller
那我們可以在個別節點 `/etc/hostname` 設定各自的名字
在 controller 節點的 `/etc/hosts` 設定對應的 IP

# 資源

- [Ansible 教學影片（YouTube）](https://www.youtube.com/watch?v=K0e7tjh3NFk&list=PLfQqWeOCIH4BDoRx8lpXXl4hqSD4GSDU5)
- [iThelp Ansible 系列文章](https://ithelp.ithome.com.tw/articles/10185016)

# [[Inventory]]

相當於資產列表，一行代表一個節點的設定
遵守 ini 的 key-value 格式
也可以加上中括號分組（group）

```ini
// 沒分組相當於 [all]
node1 ansible_connection=ssh ansible_user=root ansible_ssh_pass=password
node2 ansible_connection=ssh ansible_user=root ansible_ssh_pass=password
```

類似正則的匹配 node1~node100

```ini
node[1:100] ansible_connection=ssh ansible_user=root ansible_ssh_pass=password
```

```ini
[web1]
# node1 是設定好的 hostname
node1 ansible_connection=ssh ansible_user=root ansible_ssh_pass=password
[web2]
node2 ansible_connection=ssh ansible_user=root ansible_ssh_pass=password
```

提取組變數（其實很少放在 ini，通常都放 [[YAML]]）

```ini
[all]
node1
node2
[all:vars]
ansible_connection=ssh
ansible_user=root
ansible_ssh_pass=password
```

# [[Ping]]

ansible 的其中一個功能，跟一般的 ping 不一樣
all 所有的節點
-m module
-i 指定文件

```console
ansible node1 -m ping -i inventory.ini
//or
ansible web1 -m ping -i inventory.ini
//all
ansible all -m ping -i inventory.ini
```

# [[SSH Key]]

在 inventory.ini 裡面我們通常不會放 ansible_ssh_pass，因為實在是太危險
那我們通常都是用 ssh key 來進行登入，在 controller 上面生成 ssh key

```console
ssh-keygen
//接著命名，假設就叫 ansible_controller
ssh-copy-id -i ~/.ssh/ansible_controller node1
//接著 node1 的 ~/.ssh/authorized_keys 就會看到 ansible_controller.pub
```

這樣就算 inventory.ini 裡面沒有 ansible_ssh_pass也沒有關係

```console
ansible node1 -m ping -i inventory.ini --private-key=/home/user/.ssh/ansible_controller
```

# [[Playbook]]

XML 格式繁瑣，數據冗餘
JSON 最流行，能代表的東西比較少
[[YAML]]（yml）利用縮排和 dash 來表示

playbooks 是 ansible 設定、部署、編排的語言，透過 [[YAML]] 來編寫

```yaml
# playbook1.yaml
- hosts: node1
  name: play-test
  tasks:
    - name: check host connection
      ping:
```

之後就可以

```console
ansible-playbook playbook1.yaml -i inventory.ini --private-key=/home/user/.ssh/
```

# 核心概念

- [[Inventory]]
- [[Playbook]]
- [[Module]]

# [[Debug]]

相當重要的一個 module，詳情請看[官方文檔](https://docs.ansible.com/ansible/latest/cli/ansible-doc.html)

# 關鍵字

- item 用來做 loop 用
- `{{ }}` 雙重大括號讀取變數用，裡面可以放 Python 語法
- `when` 條件判斷（與 Jinja2 的 `or` 一起使用）

# 基本功能

- with_items 提供 loop 變數
- with_nested 提供 nested loop 變數（試著印出 99 乘法表）
- var 宣告變數
- var_files 變數的檔案
- when 條件語句
- become 使用 root 身分
- gather_facts 獲取主機訊息，ansible 默認打開，這些訊息可以當成變數來獲取

# 資料夾結構

ansible 會透過資料夾名來讀取資料

```
inventory/               ← 最上層的目錄（可自訂名稱）
├── group_vars/         ← 群組變數目錄
│   └── all.yml
├── host_vars/          ← 主機變數目錄
│   └── node1.yml
└── inventory.ini       ← 主機清單
```

# [[Ansible.cfg]] 設定檔

會查找 4 個地方（優先級遞減）：

1. playbook.yaml 所在目錄的 ansible.cfg
2. `export ANSIBLE_CONFIG=ansible.cfg`
3. /home/user/ 底下的 ansible.cfg
4. /etc/ansible.cfg

每次都要指定參數 -i 覺得麻煩

```console
ansible-playbook playbook1.yaml -i inventory.ini
```

ansible.cfg

```ini
[defaults]
inventory = ./inventory
```

設定完之後就可以直接

```console
ansible-playbook playbook1.yaml
```

# Shell [[Module]]

```console
ansible all -m shell -a "less /www/index.html"
```

# [[Template]]

假設你使用 copy，但你不同主機所需要複製的檔案不完全一樣時，就可以使用 template，可以將其視為更高級的 copy
[[Jinja2]] 是 Python 的模板語言，裡面可以使用 Python 的 for 語法 etc.

# [[Yum]] [[Module]]

```yaml
tasks:
  - name: install git
    yum:
      name: git
      state: present
```
