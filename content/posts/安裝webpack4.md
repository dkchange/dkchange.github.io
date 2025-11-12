---
title: 安裝webpack4
date: 2019-10-30 22:45:46
tags:
- webpack
- loader
- plugin
categories: [前端路線,bulid tool,Module Bundler]
---
# 前言
webpack是前端各大框架必備工具，用來編譯和打包，雖然各大場都有提供設訂好的工具，但為了理解，還是必學

# 用npm安裝webpack
使用nodejs內建的npm來安裝

    npm install -g webpack
    npm install -g webpack-cli
以上兩個包有相依関係，所以要-g就要一起-g，不然會找不到，btw関方不推薦全侷
接著

    webpack -v
一樣如果看到版本號就是安裝成功

# 運行webpack
預設編譯src目錄底下的文件
    mkdir src
建index.js檔，在裏面寫js
    webpack
不填參數預設生產環境，編譯出來的code會看不懂，壓縮過
成功運行
    webpack --mode=development
開發環境的main.js就可讀了

# webpack配置文件
因為不想指令后面一堆參數，所以配置文件是必需的
默認`webpack.config.js`
```js
const path  = require('path');
module.exports={
    entry:'./src/index.js',
    output:{
        path:  path.resolve(__dirname,'dist'),
        filename:'main.js',
    },
    mode:'development'
}
```

# npm run指令
專案需要客制化指令給使用者
package.json
```js
{
    'script':{
        'dev':'npx webpack' //可以不用加--development這個參數了^^
    }
}
//npx預設會找node_modules底下的.bin的二進位library，沒有的話就會往全侷找
```
使用者執行
    npm run dev
編譯成功

# 結合html文件
創建index.html

```html
<script src="../dist/main.js"></script>
```

# 結合多入口html文件
一般來說，現在的網站都會把所有js壓成同一支，但你可能有多個html入口的需求

```js
module.exports={
    entry:{
        main:'./src/index.js',
        hello:'./src/hello.js'
    },
    output:{
        path:  path.resolve(__dirname,'dist'),
        filename:'[name].js',
    },
    mode:'development'
}
```
# loader
## 前言
webpack和各大工具合作的包叫作loader

## babel
ecma每年都會發展新標准，為了配合跟不上的browser，我們就必需引入babel，將code轉成舊版本的code
舊版的babel安裝名沒有 **@babel**的前綴，新版的有，主要是為了分類
那我們現在就來安裝babel

    npm install --save-dev @babel/core

## babel結合webpack

    npm install -D babel-loader

babel提供轉換的功能很多，例如箭頭涵數

    npm install --save-dev @babel/plugin-transform-arrow-functions
接著把関網的設定(https://babeljs.io/docs/en/babel-plugin-transform-arrow-functions)貼進`webpack.config.js`即可

```js
module: {
  rules: [
    {
      test: /\.m?js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: 'babel-loader',
        options: {
          //presets: ['@babel/preset-env'],
          plugins: ["@babel/plugin-transform-arrow-functions"]
        }
      }
    }
  ]
}
```
或是`.babelrc`
```js
{
  "plugins": ["@babel/plugin-transform-arrow-functions"]
}
```

但是每次作新專案都要選plugin是很煩的事，所以有人作好了presets(https://babeljs.io/docs/en/presets)

    npm install --save-dev @babel/preset-env

```js
module: {
  rules: [
    {
      test: /\.m?js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: [['@babel/preset-env',{'debug':true}]],//debug可以列出所有引入的plugin
        }
      }
    }
  ]
}
```

或是`.babelrc`
```js
{
  "presets": [['@babel/preset-env']]
}
```

## polyfill
babel的功能是轉換語法   e.g.箭頭函數轉換，
polyfill是加強功能   e.g.加入promise這個api
各司其職來應付舊瀏覽器

    npm install @babel/polyfill

在此說明一下，polyfill是運行時會執行的code，所以要放在產品依賴下
而且要記得壓縮進js

```js
module.exports={
    entry:{
        main:['@babel/polyfill','./src/index.js'],
    },
    output:{
        path:  path.resolve(__dirname,'dist'),
        filename:'[name].js',
    },
    mode:'development'
}
```
或是

```js
module: {
  rules: [
    {
      test: /\.m?js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: [['@babel/preset-env',{'debug':true,'useBuiltIns':'entry'}]],//useBuiltIns應用在entry上，然后在js裏面import
        }
      }
    }
  ]
}
```

在index.js裏面import
```js
//index.js
import '@babel/polyfill'
```

那能否按需加載?不然編譯后的檔案太大
```js
    presets: [['@babel/preset-env',{'debug':true,'useBuiltIns':'usage'}]],//useBuiltIns改成usage就能按需加載了

```

## polyfill的缺點
polyfill本身是透過全侷變量的方式來添加api
這樣會汙染全侷，可能會和其他plugin形成冲突

## runtime
用來取代polyfill的，有sanbox機制，不會汙染全侷

    npm install @babel/runtime
    npm install @babel/plugin-transform-runtime
    npm install @babel/runtime-corejs2

```js
 [
    "@babel/plugin-transform-runtime",
    {
    "absoluteRuntime": false,
    "corejs": 2,
    "helpers": true,
    "regenerator": true,
    "useESModules": false,
    "version": "7.0.0-beta.0"
    }
]
```


# plugin
## 前言
plugin就是爾外功能的添加
## js檔添加hash

```js
module.exports={
    entry:{
        main:'./src/index.js',
        hello:'./src/hello.js'
    },
    output:{
        path:  path.resolve(__dirname,'dist'),
        filename:'[name]-[hash].js',//hash值會隨文件改變而變動
    },
    mode:'development'
}
```
## 動態生成html
因爲不想每次生成hash要手動更改html(https://github.com/jantimon/html-webpack-plugin)

    npm install html-webpack-plugin -D

```js
const HtmlWebpackPlugin = require('html-webpack-plugin')
module.exports={
    plugins: [
        new HtmlWebpackPlugin(
            {
                title: 'My App',
                filename: './public/index.html',
                hash:true
            }
        )
    ]
}
```
```html
<title><%= htmlWebpackPlugin.options.title %></title>
```
因爲此plugin預設webpack裝在local，所以報錯Cannot find module 'webpack/lib/node/NodeTemplatePlugin'

修改環境變量

[windows](https://stackoverflow.com/questions/9587665/nodejs-cannot-find-installed-module-on-windows)
    set NODE_PATH= /usr/lib/node_modules
mac or linux
    export NODE_PATH="/usr/lib/node_modules" 

或執行時
```js
//windows
"scripts": {
    "dev": "set NODE_PATH=/usr/lib/node_modules npx webpack"
}
//mac
"scripts": {
    "dev": "NODE_PATH=/usr/lib/node_modules npx webpack"
}
```

那最正常的作法，local裝一下就行

    npm install webpack webpack-cli -D



