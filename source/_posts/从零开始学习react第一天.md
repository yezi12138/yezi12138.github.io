---
title: 从零开始学习react第一天
date: 2017-08-10 01:17:17
tags: react
---
# 从零开始学习react第一天
------

## 从零开始学习react，我会详细地解析每一步的意义，希望能够对大家学习react有所帮助

## 目录
### &sect; [安装配置](#dev)
### &sect; [运行一个简单的react页面](#simplePage)
### &sect; [自动刷新](#refresh)


## <a name='dev'>1. 安装配置</a>

### 第一步：Node.js
首先需要到 http://nodejs.org/ 下载安装最新版本的 Node.js
我使用的版本是 node 8+ 

### 第二步：新建空前端项目
在你新建的文件夹根目录下，进入命令行，输入
> npm init

然后一路 `enter` 就行
在根目录下创建 `index.html`

    <!DOCTYPE html>
        <html lang="en">
        <head>
          <meta charset="UTF-8">
          <title>react-demo</title>
        </head>
        <body>
          <div id="app"></div>
        </body>
    </html>
在根目录下创建 `src` 文件夹，同时在里面创建 `index.js` 文件，这是我们项目的入口

整个项目结构如下：
├── package.json      // 项目配置信息 
├── index.html      // 入口 HTML  
├── dist            // dist 目录放置编译过后的文件文件
└── src             // src 目录放置源文件
    └── index.js    // 入口 js 
    
    
---

## <a name='simplePage'>运行一个简单的react页面</a>
### 第一步： 安装依赖
要运行react页面，我们需要两个包 `react` `react-dom`
> npm install react react-dom --save-dev

### 第二步：编写index.js文件

    var React = require('react')
    var ReactDom = require('react-dom')
    ReactDom.render(
      <h1>hello world</h1>,
      document.getElementById('app')
    )
    
在index.html中引用index.js

    <script type="text/javascript" src="./index.js"></script>
    
当然，如果仅仅是这样，是完全不可能运行得了页面的，因为浏览器根本不支持jsx语法，这个时候我们就要用到webpack来打包我们的页面啦
备注：这次是主要达到webpack+ES6+react项目，所以在这里不用Browser.js来转换jsx语法啦


### 第三步：配置webpack

> npm webpack --save-dev

在根目录下创建webpack.config.js文件，详细代码如下

    var webpack = require('webpack')
    // 这个plugin是用来将js注入html模板
    var HtmlwebpackPlugin = require('html-webpack-plugin')
    var path = require('path')
    
    module.exports = {
      devtool: 'eval-source-map',
      /*
       *入口文件的设置
       *支持三种形式 对象， 字符串， 数组
       *同时支持多个入口
       */
      entry: {
        app: './src/index.js'
      },
      // 输出设置
      output: {
        path: __dirname + '/dist',
        filename: 'bundle.js',
        publicPath: '/dist'
      },
      resolve: {
        // 打包后缀为wxtensions的文件
        extensions: ['.js', '.jsx'],
        // 这个是路径的简写 src == ./src
        alias: {
          'src': path.resolve(__dirname, '/src')
        }
      },
      plugins: [
        new HtmlwebpackPlugin({
         template: path.resolve(__dirname, './index.html'),
         // 把入口文件注入到html的body之前
         inject: true
        })
      ]
    }
    
这样子我们就设置了项目的入口和出口,还有模板html，<font color='blue'>我们还需要对jsx语法进行转换，才可以达到页面的展示</font>

### 第四步：运用babel转换jsx
首先我们需要下载依赖
>  npm install  babel-core babel-loader babel-preset-es2015 babel-preset-react --save-dev

`babel-preset-es2015` `babel-preset-react` 是babel的预处理模块

在package.config.js中增加一个对象：

    module: {
        loaders: [
          {
            test: /\.jsx?$/,
            loader: 'babel-loader',
            // exclude不转换指定目录， include转换
            exclude: path.resolve(__dirname, '/node_modules'),
            query: {
                presets: ['es2005', 'react']
            }
          }
        ]
    }
这样子就可以将ES6 + jsx转化为ES5 + js了
当然还有个更优雅的写法，把 `query` 去掉，在项目根目录下新建 `.babelrc` 文件，在里面写上：

    {
    "presets": ["react", "es2015"]
    }
达到的效果是一样的，你甚至还可以在 `.babelrc` 里面设置更多的属性

### 第五步：打包项目，运行页面
做到这一步已经离成功不远了，我们在根目录下进入命令行，输入：
>webpack

然后系统就开始打包啦！，我们只要等待打包结果出来就行
![webpack][1]

可以看到在 `dist` 文件夹下多出来两个文件 `bundle.js` `index.html`
我们打开 dist/index.html，可以看到我们的页面已经正常运行了
![index][2]

---

## <a name='refresh'>自动更新</a>
想象一个场景，你给app的背景修改边距，你每改一次代码都要用webpack打包一次，这样效率太低，也不利于调试和修改

这是我们可以用 `webpack-dev-server` 这个插件来实现热更新

### 第一步:万年不变的安装依赖
> npm install webpack-dev-server --save-dev

`webpack-dev-server` 是一个小型的Node.js Express服务器,它使用webpack-dev-middleware来服务于webpack的包,除此自外，它还有一个通过Sock.js来连接到服务器的微型运行时.

然后我们在 `pakage.json` 文件里面找到 `scripts` ,添加这样一段：

    "scripts": {
        "dev": "webpack-dev-server --inline"
    }

在pakage.json中加入这段代码：

    devServer: {
        inline: true,//实时刷新
        historyApiFallback: true
    }

#### <font color='red'>踩坑，请注意了：</font>  
1. pakage.json 要设置 `publicPath` : `/` 不然无法热更新
2. pakage.json 要在`plugins`设置一个新的插件 `new webpack.HotModuleReplacementPlugin()` 不然启动之后会无法显示出页面
其中：
> `--inline` 启用inline模式 详细参数可以点击链接查看[webpack-dev-server详解][3]
> `--hot` 不要按网上教程加上  webpack 2+之后加了之后就无法热更新
> `colors` 这个也不要加，会报错

### 第二步:启动
> npm run dev
然后项目就会自动打包，同时我们打开 `http://localhost:8080`

就可以看到我们的页面，改动index.js的内容，你可以看到页面会自动刷新！
这样，我们今天所要讲的内容就全部到此结束啦

by the way， 听说 `webpack-dev-server` 已经停止维护了，改为使用 `react-transform`
明天我再详细了解了解~


  [1]: http://wx1.sinaimg.cn/mw690/a359ab18gy1fidyk98xlmj20o309ut8y.jpg
  [2]: http://wx3.sinaimg.cn/mw690/a359ab18gy1fidyn2eyedj20je07qjri.jpg
  [3]: https://segmentfault.com/a/1190000006964335