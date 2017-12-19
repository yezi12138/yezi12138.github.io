---
title: 从零开始学习react第三天
date: 2017-08-11 17:47:01
tags: react
---

# 从零开始学习react第三天

------

前面我们已经配置出一个基本的react-cli，但是依然还没有完善好，因为现在的配置仅仅是打包的js和html格式的文件，对于图片，css，字体这些文件还无法进行打包处理，今天我们就开始给项目添加这些功能。

## 第一种：内联样式

### 首先按我们用index.js作为解说

    import React from 'react';
    import { render } from 'react-dom';
    if (module.hot) {
      module.hot.accept();
    }
    render(
      <div>
      <h1>hello world</h1>
      </div>,
      document.getElementById('app')
    );
    
在react中定义样式：

    const styleIndex = {
        fontSize: "14px",
        // react不推荐这种写法
        "padding-top": "20px"
    }
    
在 `index.js` 文件中写入

    import React from 'react';
    import { render } from 'react-dom';
    if (module.hot) {
      module.hot.accept();
    }
    const styleIndex = {
      // ...
    }
    render(
      <div style={styleIndex}>
      <h1>hello world</h1>
      </div>,
      document.getElementById('app')
    );
    
最后结果会将样式内联到`div`上面，这样的方式有几个特点：
1. 不会造成全局污染
2. css写法和传统写法不同
3. 嵌入的方式是内联，不利于修改
4. 无法写动画 / 伪类

### 这种写法支持react表达式
> backgroundColor: (this.state.isPadding) ? "#ccc" : "#ddd"
注意这里用的是圆括号

    const styleIndex = {
      fontSize: "14px",
      // react不推荐这种写法
      "padding-top": (this.state.isPadding) ? "20px" : 0,
      // 支持表达式写法
      backgroundColor: (this.state.isPadding) ? "#ccc" : "#ddd",
      color: "#fff"
    }
    
## CSS module
利用 `css-loader` `style-loader` 两个module，在webpack配置，就可以在需要运用js的文件下require进来，当成对象使用

### 安装依赖
> npm install --save-dev css-loader style-loader

### 创建header.js
在`src` 目录下创建一个文件夹`components`，用来存放我们的组件
然后在`components` 下创建`header.js

    import React from 'react';
    export default class HeaderTpl extends React.Component {
      constructor () {
        super()
      }
      render () {
        return (
          <div>
            <h1>this is header</h1>
          </div>
        )
      }
    }
    
### 编写css
在 `src/css` 目录下创建一个`header.css`：

    .header h1{
      background-color: #ccc;
      color: #fff;
    }
    
### 配置webpack.base.conf.js

    {
      test: /\.css$/,
      loader: "style-loader!css-loader?modules&localIdentName=[path][name]---[local]---[hash:base64:5]"
    }
    
?modules&localIdentName=[path][name]---[local]---[hash:base64:5]
这段参数的意思是启动CSSmodule ，然后让编译后的css的名字改成`[path][name]---[local]---[hash:base64:5]`，这样可以达到限定css作用域

### 在header.js引用css

    import React from 'react';
    import headerStyle from '../css/header.css'
    export default class HeaderTpl extends React.Component {
      constructor () {
        super()
      }
      render () {
        return (
          <div className={headerStyle.header}>
            <h1>this is header</h1>
          </div>
        )
      }
    }
    
通过类似于函数的调用，就可以达到引用css样式效果，而且这里的header.css最后会被编译成 `src-css-header---header---19-MS`

### 修改index.js文件：

    import React from 'react';
    import headerStyle from '../css/header.css'
    export default class HeaderTpl extends React.Component {
      constructor () {
        super()
      }
      render () {
        return (
          <div className={headerStyle.header}>
            <h1>this is header</h1>
          </div>
        )
      }
    }
运行 `npm run dev` 查看效果

### 顺便一提，我们上面用的是 `classNmae`绑定css，如果想用 `class` 绑定的话，可以安装 `babel-plugin-react-html-attrs`
#### 在 .babelrc中配置

    {
      "presets": ["react", "es2015"],
      "env": {
       "development": {
         "presets": ["react-hmre"]
       }
      },
      plugins: ['react-html-attrs']
    }
添加 `plugins: ['react-html-attrs']` 

然后我们在header.js里面就可以这样子写：

    <div class={headerStyle.header}>
        <h1>this is header</h1>
      </div>
      
达到的效果是一样的！ 也更贴近我们的日常使用习惯

## 另外一种调用css的方式
首先还原配置

### 配置webpack.base.conf.js

    {
      test: /\.css$/,
      loader: "style-loader!css-loader"
    }


### 修改`header.css`：

    .App-header{
      background-color: #ccc;
      color: #fff;
    }
    
### 在header.js引用css

    import React from 'react';
    import '../css/header.css'
    export default class HeaderTpl extends React.Component {
      constructor () {
        super()
      }
      render () {
        return (
          <div className="App-header">
            <h1>this is header</h1>
          </div>
        )
      }
    }

这种写法无法避免全局污染

## 配置sass

> npm install sass-loader node-sass --save-dev

### 在 `webpack.base.conf` 下面加一个loader

    {
      test: /\.scss$/,
      loaders: ['style-loader', 'css-loader', 'sass-loader'],
    }
    
这样子就行了