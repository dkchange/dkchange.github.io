---
title: 使用git
date: 2019-12-20 12:07:16
tags:
  - git flow
  - 版本控制
categories: 任何路線
---

# 前言

版本控制在開發過程占很重要的地位，那團隊裡該如何多人協作?整個專案管理的流程應該要怎樣才不會一天到晚衝突，功能互蓋?
參考：為你自己學 git https://gitbook.tw/ 非常推荐，基本上工作需要的东西在**為你自己學 git**都可以找到解答
https://zlargon.gitbooks.io/git-tutorial/content/file/ignore.html
https://zlargon.gitbooks.io/git-tutorial/content/file/recover.html

# 約定俗成

請儘量不要在已經 Push 出去之後再修改提交，否則可能會造成其它人的困擾
Git 在計算、產生物件的時候，是根據「檔案的內容」去做計算的，所以光是新增一個目錄，Git 是沒辦法處理它的。慣例上可以放一個名為 “.keep” 或 “.gitkeep” 的空檔案，讓 Git 能「感應」到這個目錄的存在

切好每一次的功能，有空就提交，但前提示提交的 code 都是可以 work 的，獨立且可正常運作的 code

# 免帳密 ssh key 登入

https://aben20807.blogspot.com/2018/03/1070302-git-push-ssh-key.html

# git 基本指令

## git init

## git status

## git add

檔案放到暫存區（Staging Area）

    $ git add *.html

## git commit

把暫存區的東西存放到儲存庫（Repository）裡

### 使用情境

能增加新 add 到暫存區的檔案，和修改提交訊息，但只能修改前一次的提交
\$git commit --amend -m "Welcome To Facebook"

## git log

只 show 一行資訊和版本路線
\$ git log --oneline --graph

### 使用情境

找某個人或某些人的 Commit
https://gitbook.tw/chapters/using-git/log.html
特定檔案的 Commit 紀錄
$ git log welcome.html
詳細內容
$ git log -p welcome.html

## git rm

不需要再自己 add 一次

### 使用情境

https://gitbook.tw/chapters/using-git/rename-and-delete-file.html
不是真的想把這個檔案刪掉，只是不想讓這個檔案再被 Git 控管了
\$ git rm welcome.html --cached

## git mv

改檔名，不需要再自己 add 一次
\$ git mv hello.html world.html

## git rebase

修改歷史

怎麼取消 rebase
\$ git reset ORIG_HEAD --hard

### rebase 的好處

https://youtu.be/JwqDryn8anY
rebase 會修改到歷史紀錄，所以不會像 merge 一樣按照順序，但是時間順序對於整個專案相對不是那麼重要，重要的是你修改了什麼，不要跟別人的帶買時間交互重疊，裁讓人好做 code review ，除非你的 merge 只有一個提交

## git reset

撤掉 commit
分三種情況 --mixed --soft --hard

### 使用情境

不小心使用 hard 模式 Reset 了某個 Commit，救得回來嗎
git reflog
當 HEAD 有移動的時候，Git 就會在 Reflog 裡記上一筆
\$ git reset e12d8ef --hard
就再 reset 回來一次即可

## git ignore

檔案不想讓它進到 Git 裡

### 使用情境

檔案，完全符合被忽略的規則,但沒效果？
代表該檔案，在 .gitignore 之前就存在了。.gitignore 檔案設定的規則，只對在規則設定之後的有效
\$ git rm --cached

## git blame

某個檔案的某一行是誰寫的
\$ git blame index.html

## git checkout

取分支
取檔案
會拿距離現在兩個版本以前的那個 welcome.html 檔案來覆蓋現在的工作目錄裡的 welcome.html 檔案
\$ git checkout HEAD~2 welcome.html

## git branch

分支改名字
$ git branch -m cat tiger
刪除分支
$ git branch -d dog
還原刪除的分支
\$ git branch new_cat b174a5a

## git stash

檔案暫存起來
\$git stash list

\$git stash pop

## git cherry-pick

撿別的分支的 Commit 過來合併
git cherry-pick 6a498ec
加上 --no-commit 參數，撿過來的 Commit 不會直接合併，而是會先放在暫存區
\$ git cherry-pick 6a498ec --no-commit
再挑想要的檔案合併

從別的分支撿檔案過來，
https://gitbook.tw/chapters/faq/cherry-pick.html

git cherry-pick 6a498ec

## git squash

改寫提交，強制提交上 gitHUB
https://yodalee.blogspot.com/2014/04/git-squash-commit.html
https://backlog.com/git-tutorial/tw/stepup/stepup7_5.html


## git fetch

取遠端分支
git fetch origin feature/ashui/weifu_payment

## git flog

### 使用情境
你不小心git pull了，只看git log 還原不回去，你想要撤銷這個動作

运行git reflog命令查看你的历史变更记录
然后用git reset --hard HEAD@{n}，（n是你要回退到的引用位置）回退。

或是 git reset --hard 40a9a83


# gitflow 是其中一種工作流程

gitflow 是其中一種工作流程
https://gitbook.tw/chapters/gitflow/why-need-git-flow.html


## GUI 工具

那由於 gitflow 學習難度稍高一些，要團隊一起實踐比較困難，所以借助工具 smartGit，可以幫我們省下很多功夫
https://youtu.be/ualXHytifbg

# 遭遇案例（情況）

## fatal: remote origin already exists.

是因為你那到一個包含.git 的檔案，裡面已經有人 remote 的 url，此時要刪掉時
git remote add origin url
fatal: remote origin already exists.
git remote rm origin
此時就可以再下
git remote add origin url

## git replacing LF with CRLF

If you are a single developer working on a windows machine, and you don't care that git automatically replaces LFs to CRLFs, you can turn this warning off by typing the following in the git command line
git config core.autocrlf true

## 查看當前檔案和前一次提交的差別

git diff HEAD path/to/file

## 更新專案

git pull origin branch_name

## 還原檔案

git checkout path/to/file

## 切換分支

git checkout branch_name

## 建立新分支並切換進去

git checkout -b new_branch_name

## show 上一次提交

git show

git push

## 取消 git add

git reset HEAD [file_name]

## git 忽略檔案

檔案，在 .gitignore 之前就存在了。.gitignore 檔案設定的規則，只對在規則設定之後的有效，那些已經存在的檔案就像既得利益者一樣，這些規則是對他們沒有效果的
如果想套用 .gitignore 的規則，就必須先使用 git rm --cached 指令把這些既得利益者請出 Git



## 題交錯分支了

git reset HEAD^
git log
這樣看 log 就會發現提交被清掉了


## 使用 Git 時如何做出跨 repo 的 cherry-pick

https://blog.m157q.tw/posts/2017/12/30/git-cross-repo-cherry-pick/

## git patch apply

https://youtu.be/sPJkfYvdfIE

使用情境：東西作到一半需要別人幫你繼續做，這時候就可以存成 patch

## git 佈署環境

https://www.youtube.com/watch?v=NFsIuTFd6bA&feature=youtu.be&list=PLwAKR305CRO_WMJAJxGz6UCViQyAjdjgo&t=599
https://bolerolily.github.io/2018/08/08/%E5%88%A9%E7%94%A8Git-Hooks%E4%B8%8Ersync%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2/

## git commit -m 加备注时如何换行

先输入双引号，但是不要输入后面的双引号，然后就可以输入多行了

# github git clone

https://serverfault.com/questions/447028/non-interactive-git-clone-ssh-fingerprint-prompt
https://superuser.com/questions/421997/what-is-a-ssh-key-fingerprint-and-how-is-it-generated

## 空資料夾

如前言所說
當我們想要空資料夾，被提交到 git，我們必須在空資料夾底下新增.gitignore

```console
//.gitignore
*
!.gitignore

```
## git GUI 向下的箭頭

有的時候會在GUI軟體看到一些向下的箭頭，沒別的意思，就只是因為再拉一條線會太長，
為了方便顯示，它給個向下的箭頭可以展開，所以那個箭頭是可以按的

[带你一步一步看懂Git图谱](https://juejin.cn/post/6844903669758951432)

## 如何清理git的版本庫

[Cleaning up a git repo for reducing the repository size](https://medium.com/@sangeethkumar.tvm.kpm/cleaning-up-a-git-repo-for-reducing-the-repository-size-d11fa496ba48)
```console
# 先複製出一份沒有歷史紀錄的分支
git checkout — orphan latest_branch
git add -A
git commit -am “Initial commit message” #Committing the changes
git branch -D master #Deleting master branch
git branch -m master #renaming branch as master
git push -f origin master #pushes to master branch
git gc — aggressive — prune=all # remove the old files
```

## git stash abort
[How to abort a stash pop?](https://stackoverflow.com/questions/8515729/how-to-abort-a-stash-pop)
```console
git reset --merge
```