---
title: 使用npm
date: 2019-10-31 23:53:26
tags: 
- npm
categories: [前端路線,package manager]
---
# 安裝nodejs
windows環境去官網下載安裝檔安裝

    node -v
如果成功看到版本號就是安裝成功

# nodejs 版本問題
建議安裝nvm，windows就裝nvm-windows
指令都一樣，只是windows記得在CMD底下運行
情境:
What's your nodejs version? jsdom requires node >= 10.
node -e "try {} catch {}"
node -v

# npm常用指令
初始化一個專案

    npm init "專案名"
然後就會看到多了個package.json檔，那如果你是一個現成的專案，你就會看到這隻檔案

    npm install -g "package的名字"
上面-g代表global**全局安裝**，不管你切到哪個資料夾直接下這個指令，windows系統都找的到，通常是各個專案都有機會用到的工具

    npm install "package的名字"
這就直接安裝在專案底下，**本地安裝**會出現在node_modules資料夾，來方便專案引用，所以專案要引用所有module的**根目錄**就是在這

    npm install --save-dev "package的名字" 
一樣直接安裝在開發依賴底下也可以用-D，注意是大寫D，**本地安裝**會出現在node_modules資料夾，通常是開發時才會用到的工具，生產環境的code不會引用到，所以大致上你在開發專案時，為了讓人家順利使用，很少用-g這種全局的安裝
https://stackoverflow.com/questions/44764004/ts-node-is-not-recognized-as-an-internal-or-external-command-operable-program

    npm uninstall --save webpack
加上--save移除時才會順便更新package.json檔

    npm update "package的名字"
更新某package

# 指定版本號
    nodejs採用semver版本號規範，這版本號的命名規則再去看看

    npm install webpack@^1.*
符號號代表什麼再上網查語意吧

# npm run
在package.json檔裡面有個屬性叫作scripts，裡面可以對這個專案自訂指令，簡單的說就是讓你把一長串系統指令讓你取個別名，不用打那長串系統指令
`寫指令時記得要考慮使用者的系統環境，linux和windows命令不一樣`

```js
{
    "scripts": {
        "clean": "rm -r dist && mkdir dist"
        }
}

```
像上面這樣，使用者要清空dist(通常這是發佈到生產環境的資料夾)資料夾時，只要在命令行下npm run clean

# 檢查

    npm list --depth=0
    列出第一層的包，有依賴錯誤顯示出來

    情境:
    npm ERR! peer dep missing: canvas@^2.5.0, required by jsdom@15.2.1
    Just upgrade your npm version to the latest.
canvas is an optional peer dependency of jsdom and npm has supported this feature since 6.11.
    npm install npm@latest -g

    npm install 
    npm list --depth=0
    重裝後搞定


