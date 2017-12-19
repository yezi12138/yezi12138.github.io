---
title: 从零开始学习react第五天
date: 2017-08-24 18:27:24
tags: react
---
# 从零开始学习react第五天

------

第五天，这次学习如何使用react-router

## 安装 `react-router`
> npm install react-router --save

### 踩坑点： 这里我用的是2.x版本，如果使用4.x版本，下面方法就不支持了

## 项目加入router
新建 `root.js`， 代码如下：

    import React from 'react';
    import {Router, Route, hashHistory, IndexRoute} from 'react-router'
    import App from './components/app'
    import List from './components/list'
    import Child from './components/child'
    export default class Root extends React.Component {
      render () {
        return (
          <Router history={hashHistory}>
            <Route path="/" component={App} />
            <Route path="/list" component={List}>
              <IndexRoute component={Child} />
              <Route path="/child" component={Child} />
              <Route path="/App" component={App} />
            </Route>
          </Router>
        )
      }
    }
    
1. `Router` 定义一个路由
2. `history={hashHistory}` 定义路由路径
3. `path` 定义路由路径
4. `IndexRoute` 进入路由list时，默认显示该子路由，相当于子路由默认页，如果不加这个，在进入list时候并不会显示路由

### 踩坑点： 
1. 这里的写法是 `path="/child"` 这样可以通过url `xxxx.com/child` 访问到页面，但是如果这样写 `path="child"` 把 `/` 去掉，就不能通过路由访问到了，但是也不会报错，把他设置成子路由首页也是没问题的
2. 上面代码中，用户访问/child时，会先加载list组件，然后在它的内部再加载child组件

### 新增list.js，代码如下：

    import React from 'react';
    export default class List extends React.Component {
      render () {
        return (
          <div>
            <div>this is list</div>
            {this.props.children}
          </div>
        )
      }
    }
    
`{this.props.children}` 这里的代码是为了显示子路由，如果不加上这个，子路由就无法显示

然后就可以运行整个项目，路由搭建成功

## Link 跳转路由
Link组件用于取代<a>元素，生成一个链接，允许用户点击后跳转到另一个路由。它基本上就是<a>元素的React 版本，可以接收Router的状态

    import {Link} from 'react-router';
    render () {
        return (
            <div>
                // ...
                <Link to='/list' activeStyle={{color: 'red'}}>to list</Link>
                <Link to='/child' activeClassName="active">to child</Link>
            </div>
        )
    }
    
`activeStyle` 使用Link组件的activeStyle属性添加行内样式
`activeClassName` 使用Link组件的activeClassName属性添加css样式

### 在Router组件之外的跳转
在Router组件之外，导航到路由页面，可以使用浏览器的History API，像下面这样写：

    import { browserHistory } from 'react-router';
    browserHistory.push('/some/path');
    

踩坑点：使用`browserHistory` 刷新浏览器会找不到页面
原因是使用router,改变的url路径仅仅是在本地，他并没有向服务器发出请求。以往传统的网站都是向服务器发出请求，然后获取返回的页面。
解决方法： 使用 `hashHistory` ，因为他用了 `hash tag` 即浏览器url的#后面内容
比如 `host/#/path/params`  `/path/params`是hash，这样不管你刷新多少次，都是在想服务器请求主页内容，这样就不会返回错误页，同时在本地客户端，借用hash tag进行router跳转
    
更多用法请查阅这里 [React Router 使用教程][1]


  [1]: http://www.ruanyifeng.com/blog/2016/05/react_router.html?utm_source=tool.lu