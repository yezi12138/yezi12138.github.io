---
title: 从零开始学习react第二天
date: 2017-08-10 21:56:05
tags: react
---

# 从零开始学习react第二天

------

## 前言

首先我们先完善第一天留下来的问题，之前我们用 `babel-loader` `babel-preset-react` `babel-preset-es2005` 三个包将ES6语法转换为ES5，相对于大多数浏览器，这样的设置可能已经足够了，但是对于少数不支持ES5语法的浏览器，这样的设置可能还不够，而且 `babel-loader` 对于ES6的 `promise` `fetch`，ES5的新方法 `Object.assign` ,虽然能正确运行，但是对于IE9以下的浏览器来说并不能兼容
这个时候就需要 `polyfill`

为了能随心所欲地使用ES6语法，这里我推荐使用  `babel-polyfill` `babel-preset-env`
关于 `babel-preset-env` 的选择原理看这里：
[21 分钟精通前端 Polyfill 方案][1]
[babel-polyfill和babel-runtime对比][2]

### 安装  `babel-polyfill`
> npm install babel-polyfill --save

### 修改pakage.json配置，让 `babel-polyfill` 单独插入到html
为了实现多入口的注入，我们用到 `new webpack.optimize.CommonsChunkPlugin(options)`
代码如下：

    plugins: [
        // ...
        new webpack.optimize.CommonsChunkPlugin({
            name: 'verdor',
            filename: 'polyfill.js'
        }) 
    ]
    // 修改入口文件,增加vendor
    entry: {
        app: './src/index.js',
        verdor: 'babel-polyfill'
    }
    
这样运行webpack，可以看到编译后的html下面有两个script文件引用了

---

回到正题，今天学习的是`babel-plugin-react-transform`的热加载

## babel-plugin-react-transform 热加载的实现
之前我们已经实现了页面的自动刷新，用的命令是：
> webpack-dev-server --inline

原本想用 `react-hot-loader` 可是官方已经停止维护，推荐我们使用 `babel-plugin-react-transform`

### 于是我们先安装包
> npm install --save-dev babel-plugin-react-transform

还有一些转换器搭配使用
> npm install --save-dev react-transform-hmr
> npm install --save-dev react-transform-catch-errors redbox-react

`babel-plugin-react-transform` 这是个基本的架子
`react-transform-hmr` 是真正实现热更新的插件
`react-transform-catch-errors` `redbox-react`  是实现在页面上显示错误，不用每次都打开控制台,可以不安装

要让新建的两个transform生效,只需再安装一个present
> npm install babel-preset-react-hmre --save-dev

### 第一次启动
> npm run dev

发现结果并不是和我们想象中一样，达到了热刷新，结果还是和以前一样，全面刷新。
于是我各种百度，还是不得其解，只好去他们的官网下载了一个[demo][3]

发现，他们的demo并不是用 `webpack-dev-server` 作为服务器，而是用 `webpack-dev-middleware` 和 `webpack-hot-middleware` 开启热刷新

于是我开始仿照他们的demo，重构了我的目录结构

### 重构项目结构

安装这两个模块
> npm install --save-dev webpack-dev-middleware webpack-hot-middleware

然后新建一个文件夹build，<font color='blue'>把webpack.config.js放进去并且改名为webpack.base.conf.js</font>，然后新建两个文件 `webpack.dev.conf.js` `webpack.prod.conf.js`

目录结构如下：
![目录结构][4]

#### `webpack.dev.conf.js` 文件主要是在开发环境所要用到的配置，代码如下：

    var webpack = require('webpack')
    var merge = require('webpack-merge')
    var HtmlwebpackPlugin = require('html-webpack-plugin')
    var path = require('path')
    var config = require('./webpack.config')
    module.exports = merge(config, {
      // cheap-module-eval-source-map is faster for development
      devtool: '#cheap-module-eval-source-map',
      plugins: [
        new webpack.HotModuleReplacementPlugin(),
        new webpack.NoEmitOnErrorsPlugin(),
        new HtmlwebpackPlugin({
        template: path.resolve(__dirname, '../index.html'),
        inject: true
        }),
      ]
    })

这里必须用到 `new webpack.HotModuleReplacementPlugin()` ，不然无法启用热更新
主要把仅在开发过程用到的配置写在这里，然后合并`webpack.base.conf.js`的参数

#### `webpack.base.conf.js` 的配置如下：

    var path = require('path')
    module.exports = {
      devtool: 'eval-source-map',
      entry: {
        index: './src/index.js'
      },
      output: {
        path: __dirname + '/dist',
        filename: 'bundle.js',
        publicPath: '/'
      },
      resolve: {
        extensions: ['.js', '.jsx'],
        alias: {
          'src': path.resolve(__dirname, '/src')
        }
      },
      module: {
        loaders: [
        {
          test: /\.jsx?$/,
          loader: 'babel-loader',
          exclude: path.resolve(__dirname, '/node_modules')
        }
        ]
      }
    }

这里只留下entry，resolve，output三个属性

#### 在 `webpack.dev.conf.js` 中添加这么一段代码，将 `webpack-hot-middleware` 注入到entry里面去

    Object.keys(config.entry).forEach(function (name) {
      config.entry[name] = ['webpack-hot-middleware/client'].concat(config.entry[name])
    })
    
当然你也可以直接在 `webpack.base.conf.js` 中的entry里面这样写：

    entry: [
        webpack-hot-middleware/client,
        './src/index.js'
    ]
但是为了开发和生产环境分离，我们还是推荐选择第一种写法，<font color='red'>注意第一种写法entry是对象，而第二种写法则是数组</font>

## 创建dev-server.js文件，开启服务
在 build 文件夹下面创建 `dev-server.js`
代码如下：

    var path = require('path')
    var express = require('express')
    var webpack = require('webpack')
    var config = require('./webpack.dev.conf')
    var app = express()
    var compiler = webpack(config)
    app.use(require('webpack-dev-middleware')(compiler, {
      noInfo: true,
      publicPath: config.output.publicPath
    }))
    app.use(require('webpack-hot-middleware')(compiler))
    app.listen(3000, function(err) {
      if (err) {
        console.log(err)
        return
      }
      console.log('Listening at http://localhost:3000')
    })

这个文件引进 `webpack.base.conf.js` 的配置，用express开启了一个服务器，监听端口3000

修改script中的配置：

    "scripts": {
     "dev": "node build/dev-server.js"
    }

然后命令行输入： npm run dev
等待几秒，你就会发现我们的页面正常显示了！

### 热刷新
问题来了，当你修改index.js 里面的值的时候，你会发现页面根本没有自动刷新，而且控制台输出了这样一段信息
> following modules couldn't be hot updated: (Full reload needed)
This is usually because the modules which have changed (and their parents) do not know how to hot reload themselves.

这是因为我们仅仅是运用了 `webpack-dev-middleware` 和 `webpack-hot-middleware`
一开始说过的 `babel-plugin-react-transform` 这三个包还没有用上
怎么用呢？我们可以直接在 `.babelrc` 中添加：

    {
      "presets": ["react", "es2015"],
      "env": {
       "development": {
         "presets": ["react-hmre"]
       }
      }
    }
这样子就行了，然后重启一次服务，这样子就可以实现热刷新了！

### 8月12日更新
对于文件入口，要加入以下代码: 

    if(module.hot)
     {
         module.hot.accept();
     }

不然对于主入口的数据无法进行热更新，当更改主入口数据时候会报错
> following modules couldn't be hot updated: (Full reload needed)
This is usually because the modules which have changed (and their parents) do not know how to hot reload themselves.


  [1]: https://segmentfault.com/a/1190000010106158
  [2]: https://segmentfault.com/q/1010000005596587
  [3]: https://github.com/gaearon/react-transform
  [4]: http://wx1.sinaimg.cn/mw690/a359ab18gy1fieycd93xaj20he08jaa6.jpg