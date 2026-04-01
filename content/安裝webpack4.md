---
title: 安裝webpack4
date: 2019-10-30 22:45:46
tags:
  - webpack
  - loader
  - plugin
categories:
  - 前端路線
  - build tool
  - Module Bundler
---

# 前言

[[Webpack]] 是前端各大框架必備工具，用來編譯和打包，雖然各大框架都有提供設定好的工具，但為了理解原理，還是必學。

# 用 npm 安裝 Webpack

使用 [[Node.js]] 內建的 [[npm]] 來安裝：

```bash
npm install -g webpack
npm install -g webpack-cli
```

以上兩個包有相依關係，所以要 `-g` 就要一起 `-g`，不然會找不到。BTW 官方不推薦全域安裝。

接著檢查版本：

```bash
webpack -v
```

一樣如果看到版本號就是安裝成功。

# 運行 Webpack

預設編譯 `src` 目錄底下的檔案：

```bash
mkdir src
# 建立 index.js 檔，在裡面寫 JS
webpack
```

不填參數預設生產環境，編譯出來的 code 會看不懂，壓縮過。

成功運行：

```bash
webpack --mode=development
```

開發環境的 `main.js` 就可以讀了。

# Webpack 配置文件

因為不想指令後面一堆參數，所以配置文件是必需的，預設是 `webpack.config.js`：

```javascript
const path = require("path")
module.exports = {
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "main.js",
  },
  mode: "development",
}
```

# npm run 指令

專案需要客製化指令給使用者，在 `package.json` 加上：

```json
{
  "scripts": {
    "dev": "npx webpack"
  }
}
```

> 小知識：`npx` 預設會找 `node_modules` 底下的 `.bin` 的二進位 library，沒有的話就會往全域找。

使用者執行：

```bash
npm run dev
```

編譯成功！

# 結合 HTML 檔案

建立 `index.html`：

```html
<script src="../dist/main.js"></script>
```

# 結合多入口 HTML 檔案

一般來說，現在的網站都會把所有 JS 壓成同一支，但你可能有多個 HTML 入口的需求：

```javascript
module.exports = {
  entry: {
    main: "./src/index.js",
    hello: "./src/hello.js",
  },
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "[name].js",
  },
  mode: "development",
}
```

# npm run 指令

專案需要客製化指令給使用者
package.json

```js
{
    'script':{
        'dev':'npx webpack' //可以不用加--development這個參數了^^
    }
}
// npx 預設會找 node_modules 底下的 .bin 的二進位 library，沒有的話就會往全域找
```

使用者執行
npm run dev
編譯成功

# 結合 HTML 文件

建立 index.html

```html
<script src="../dist/main.js"></script>
```

# 結合多入口 HTML 文件

一般來說，現在的網站都會把所有 JS 壓成同一支，但你可能有多個 HTML 入口的需求

```js
module.exports = {
  entry: {
    main: "./src/index.js",
    hello: "./src/hello.js",
  },
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "[name].js",
  },
  mode: "development",
}
```

# Loader

## 前言

[[Webpack]] 和各大工具合作的包叫作 **Loader**。

## Babel

ECMA 每年都會發展新標準，為了配合跟不上的瀏覽器，我們就必須引入 [[Babel]]，將 code 轉成舊版本的 code。

舊版的 Babel 安裝名沒有 `@babel` 的前綴，新版的有，主要是為了分類。那我們現在就來安裝 Babel：

```bash
npm install --save-dev @babel/core
```

### Babel 結合 Webpack

```bash
npm install -D babel-loader
```

Babel 提供轉換的功能很多，例如箭頭函數：

```bash
npm install --save-dev @babel/plugin-transform-arrow-functions
```

接著把[官方設定](https://babeljs.io/docs/en/babel-plugin-transform-arrow-functions)貼進 `webpack.config.js` 即可：

```javascript
module: {
  rules: [
    {
      test: /\.m?js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: "babel-loader",
        options: {
          // presets: ['@babel/preset-env'],
          plugins: ["@babel/plugin-transform-arrow-functions"],
        },
      },
    },
  ]
}
```

或是建立 `.babelrc`：

```json
{
  "plugins": ["@babel/plugin-transform-arrow-functions"]
}
```

但是每次作新專案都要選 plugin 是很煩的事，所以有人作好了 [presets](https://babeljs.io/docs/en/presets)：

```bash
npm install --save-dev @babel/preset-env
```

```javascript
module: {
  rules: [
    {
      test: /\.m?js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: "babel-loader",
        options: {
          presets: [["@babel/preset-env", { debug: true }]],
          // debug 可以列出所有引入的 plugin
        },
      },
    },
  ]
}
```

或是 `.babelrc`：

```json
{
  "presets": [["@babel/preset-env"]]
}
```

## Polyfill

Babel 的功能是轉換語法（e.g. 箭頭函數轉換），[[Polyfill]] 是加強功能（e.g. 加入 Promise 這個 API），各司其職來應付舊瀏覽器。

```bash
npm install @babel/polyfill
```

在此說明一下，polyfill 是運行時會執行的 code，所以要放在產品依賴下，而且要記得壓縮進 JS：

```javascript
module.exports = {
  entry: {
    main: ["@babel/polyfill", "./src/index.js"],
  },
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "[name].js",
  },
  mode: "development",
}
```

或是在 `webpack.config.js` 設定：

```javascript
module: {
  rules: [
    {
      test: /\.m?js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: "babel-loader",
        options: {
          presets: [
            [
              "@babel/preset-env",
              {
                debug: true,
                useBuiltIns: "entry",
              },
            ],
          ],
        },
      },
    },
  ]
}
```

在 `index.js` 裡 import：

```javascript
// index.js
import "@babel/polyfill"
```

那能否按需加載？不然編譯後的檔案太大：

```javascript
presets: [
  [
    "@babel/preset-env",
    {
      debug: true,
      useBuiltIns: "usage", // useBuiltIns 改成 usage 就能按需加載了
    },
  ],
]
```

### Polyfill 的缺點

Polyfill 本身是透過全域變量的方式來添加 API，這樣會汙染全域，可能會和其他 plugin 形成衝突。

## Runtime

用來取代 polyfill 的，有 sandbox 機制，不會汙染全域：

```bash
npm install @babel/runtime
npm install @babel/plugin-transform-runtime
npm install @babel/runtime-corejs2
```

```javascript
;[
  "@babel/plugin-transform-runtime",
  {
    absoluteRuntime: false,
    corejs: 2,
    helpers: true,
    regenerator: true,
    useESModules: false,
    version: "7.0.0-beta.0",
  },
]
```

或是`.babelrc`

```js
{
  "plugins": ["@babel/plugin-transform-arrow-functions"]
}
```

但是每次作新專案都要選 plugin 是很煩的事，所以有人作好了 [presets](https://babeljs.io/docs/en/presets)

    npm install --save-dev @babel/preset-env

```js
module: {
  rules: [
    {
      test: /\.m?js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: "babel-loader",
        options: {
          presets: [["@babel/preset-env", { debug: true }]], //debug可以列出所有引入的plugin
        },
      },
    },
  ]
}
```

或是`.babelrc`

```js
{
  "presets": [['@babel/preset-env']]
}
```

## Polyfill（重複內容）

Babel 的功能是轉換語法（e.g. 箭頭函數轉換），[[Polyfill]] 是加強功能（e.g. 加入 Promise 這個 API），各司其職來應付舊瀏覽器。

    npm install @babel/polyfill

在此說明一下，polyfill 是運行時會執行的 code，所以要放在產品依賴下，而且要記得壓縮進 JS：

```js
module.exports = {
  entry: {
    main: ["@babel/polyfill", "./src/index.js"],
  },
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "[name].js",
  },
  mode: "development",
}
```

或是：

```js
module: {
  rules: [
    {
      test: /\.m?js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: "babel-loader",
        options: {
          presets: [["@babel/preset-env", { debug: true, useBuiltIns: "entry" }]], //useBuiltIns應用在entry上，然後在js裏面import
        },
      },
    },
  ]
}
```

在 index.js 裏面 import：

```js
//index.js
import "@babel/polyfill"
```

那能否按需加載？不然編譯後的檔案太大：

```js
    presets: [['@babel/preset-env',{'debug':true,'useBuiltIns':'usage'}]],//useBuiltIns改成usage就能按需加載了

```

## Polyfill 的缺點

Polyfill 本身是透過全域變量的方式來添加 API，這樣會汙染全域，可能會和其他 plugin 形成衝突。

## Runtime

用來取代 polyfill 的，有 sandbox 機制，不會汙染全域：

    npm install @babel/runtime
    npm install @babel/plugin-transform-runtime
    npm install @babel/runtime-corejs2

```js
;[
  "@babel/plugin-transform-runtime",
  {
    absoluteRuntime: false,
    corejs: 2,
    helpers: true,
    regenerator: true,
    useESModules: false,
    version: "7.0.0-beta.0",
  },
]
```

# Plugin

## 前言

Plugin 是用來擴充額外功能的。

## JS 檔添加 Hash

```javascript
module.exports = {
  entry: {
    main: "./src/index.js",
    hello: "./src/hello.js",
  },
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "[name]-[hash].js", // hash 值會隨檔案改變而變動
  },
  mode: "development",
}
```

## 動態生成 HTML

因為不想每次生成 hash 要手動更改 HTML，要安裝 [html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin)：

```bash
npm install html-webpack-plugin -D
```

```javascript
const HtmlWebpackPlugin = require("html-webpack-plugin")
module.exports = {
  plugins: [
    new HtmlWebpackPlugin({
      title: "My App",
      filename: "./public/index.html",
      hash: true,
    }),
  ],
}
```

```html
<title><%= htmlWebpackPlugin.options.title %></title>
```

因為此 plugin 預設 webpack 裝在 local，所以報錯 `Cannot find module 'webpack/lib/node/NodeTemplatePlugin'`。

### 修改環境變量

**Windows：**

```bash
set NODE_PATH=/usr/lib/node_modules
```

**Mac or Linux：**

```bash
export NODE_PATH="/usr/lib/node_modules"
```

或執行時：

```json
{
  "scripts": {
    "dev": "set NODE_PATH=/usr/lib/node_modules npx webpack"
  }
}
```

```json
{
  "scripts": {
    "dev": "NODE_PATH=/usr/lib/node_modules npx webpack"
  }
}
```

那最正常的做法，local 裝一下就行：

```bash
npm install webpack webpack-cli -D
```
