---
title: 使用ansible
date: 2020-09-10 10:39:44
tags:
categories:
---

# 前言

ansible 算是一個把yaml檔用到淋漓盡致的軟體，主要運用於同時多台自動化佈署，playbook 可以說是將yaml檔再做編譯的自創語言

假設我現在有三個節點，controller, node1, node2 我們的ansible會安裝在controller
那我們可以在個別節點`/etc/hostname`設定各自的名字
在controller節點的`/etc/hosts`設定對應的IP

# 資源
https://www.youtube.com/watch?v=K0e7tjh3NFk&list=PLfQqWeOCIH4BDoRx8lpXXl4hqSD4GSDU5

https://ithelp.ithome.com.tw/articles/10185016

# inventory.ini
相當於資產列表，一行代表一個節點的設定
遵守ini 的key-value格式
也可以加上中括號分組(group)
```ini
// 沒分組相當於[all]
node1 ansible_connection=ssh ansible_user=root ansible_ssh_pass=password
node2 ansible_connection=ssh ansible_user=root ansible_ssh_pass=password
```
類似正則的匹配 node1~node100
```ini
node[1:100] ansible_connection=ssh ansible_user=root ansible_ssh_pass=password
```

```ini
[web1]
# node1 是設定好的hostname
node1 ansible_connection=ssh ansible_user=root ansible_ssh_pass=password
[web2]
node2 ansible_connection=ssh ansible_user=root ansible_ssh_pass=password
```

提取組變數(其實很少放在ini，通常都放yaml)
```ini
[all]
node1
node2
[all:vars]
ansible_connection=ssh 
ansible_user=root 
ansible_ssh_pass=password

```
# ping 
ansible的其中一個功能，跟一般的ping不一樣
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

# ssh key
在inventory.ini裡面我們通常不會放ansible_ssh_pass，因為實在是太危險
那我們通常都是用ssh key來進行登入，在controller上面生成ssh key
```console
ssh-keygen
//接著命名，假設就叫ansible_controller
ssh-copy-id -i ~/.ssh/ansible_controller node1
//接著node1的~/.ssh/authorized_keys就會看到ansible_controller.pub
```
這樣就算inventory.ini裡面沒有ansible_ssh_pass也沒有關係
```console
ansible node1 -m ping -i inventory.ini --private-key=/home/user/.ssh/ansible_controller
```

# playbook
xml 格式繁瑣，數據冗餘
json 最流行，能代表的東西比較少
yaml yml 利用縮排和dash來表示

playbooks是ansible設定，佈署，編排的語言，透過yaml來編寫

```yaml
# playbook1.yaml
- host: node1
  name: play-test
  task:
    - name: check host connection
      ping: 
```
之後就可以
```console
ansible-playbook palybook1.yaml -i inventory.ini --private-key=/home/user/.ssh/ 
```

# 核心概念
- inventory
- playbook
- module

# debug 
相當重要的一個module，詳情請看文檔

# 關鍵字
- item 用來做loop用
- 雙重大括號讀取變數用，裡面可以放python語法
- or

# 基本功能
- with_items 提供loop變數
- with_nested 提供nested loop變數(試著映出99乘法表)
- var 宣告變數
- var_files 變數的檔案
- when 條件語句
- become 使用root身分
- gather_facts 獲取主機訊息，ansible默認打開，這些訊息可以當成變數來獲取

# 資料夾名
ansible 會透過資料夾名來讀資料
- inventory 最上層的目錄
    - group_vars
    - host_vars

# 設定檔

會查找4個地方(優先級遞減):
1. palybook.yaml所在目錄的ansible.cfg
2. export ANSIBLE_CONFIG=ansible.cfg
3. /home/user/底下的ansible.cfg
4. /etc/ansible.cfg

每次要都要指定餐參數i 覺得麻煩
```console
ansible-playbook palybook1.yaml -i inventory.ini
```

ansible.cfg
```ini
[defaults]
inventory = ./inventory
```
設定完之後就可以直接
```console
ansible-playbook palybook1.yaml 
```

# shell
```console
ansible all -m shell -a "less /www/index.html"
```

# template
假設你使用copy，但你不同主機所需要上複製的檔案不完全一樣時，就可以使用template，可以將其視為更高級的copy
jinja是python的模板語言，裡面可以使用python的for語法ect

# yum 
```yaml
task:
  - name: install git
    yum:
      name: git
      state: present
```