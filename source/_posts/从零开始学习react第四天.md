---
title: 从零开始学习react第四天
date: 2017-08-12 17:41:25
tags: react
---

# 从零开始学习react的第四天

------

## 前言：
昨天我们已经添加了对于css，图片之类的处理，那么我们现在就可以开始我们的项目了，对于完全没了解react语法的，可以先去看阮一峰老师的教程： [react入门][1]
我这里就不再重复写教程了，主要了解react怎么render，如何绑定数据，生命周期，借下来我给个简单例子说明一下，热身一下。

### 主函数 app.js

    import ComponentHeader from './header'
    render() {
        return (
          <div>
            <ComponentHeader/>
            <h1>this is mainBody page</h1>
          </div>,
          document.getElementById('app')
        )
    }
    
`ComponentHeader` 作为组件导入
注意： 导入的组件一定要大写 `ComponentHeader` 不能写成 `componentHeader`
    
### 子函数 header.js

    import React from 'react';
    export default class Header extends React.Component {
      constructor () {
        super()
        this.state = {
          title: 'this is header page'
        }
      }
      changeInfo () {
        this.setState({
          info: 'change info'
        })
      }
      render () {
        return (
          <div>
          <h1>{this.state.title}</h1>
          <input type='button' type='button' onClick={this.changeInfo.bind(this)}/>
          </div>
        )
      }
    }
    
`constructor` 是react组件的构造函数
`this.state` 设置组件的内部初始值，在`constructor` 里面定义
`super()` 初始化基类的数据
`{this.state.title}` react数据的绑定
`changeInfo` 自定义的函数，用来测试setState作用
`this.setState` 用来改变state
`export default` 是将组件导出来
`input` 注意事件绑定的方法 `bind` ，否则会报错
> Cannot read property 'setState' of null

简单地过了一遍基本知识点，接下来说说今天学习的react父子组件通信

---

## react父子组件通信

### 父向子组件传参

在父组件app.js里面：

    render() {
        return (
          <div>
            <ComponentHeader info={this.state.toHeader}/>
            <h1>this is mainBody page</h1>
          </div>,
          document.getElementById('app')
        )
    }
在要传递上面写上要传递的参数，左边 `info` 表示参数名，右边 `this.state.toHeader` 是传递参数值

在子组件header.js里面：

    export default class Header extends React.Component {
      // ...
      render () {
        return (
          <div>
          <h1>this is header page</h1>
          <p>info: {{this.props.info}}</p>
          <p>子页面输入: <input type='button' type='text' onChange={this.props.changeInput}/></p>
          </div>
        )
      }
    }

通过 `this.props.[name]` 获取传进来的数据

### 子组件向父组件传参
要求： 子组件输入数据，在父组件中显示

header.js子组件：

    render () {
        return (
          <div>
          // ...
          <p>子页面输入：<input type='text' onChange={this.props.changeInput}/> </p>
          </div>
        )
      }

app.js父组件：

    changeInput (event) {
        this.setState({
          age: event.target.value，
          title: 'this is mainBody page',
          toHeader: 'info comes from mainBody page'
        })
      }
      render() {
        return (
          <div>
            // ...
            <ComponentHeader info={this.state.toHeader} changeInput={this.changeInput.bind(this)}/>
            <p>age: {this.state.age}</p>
          </div>,
          document.getElementById('app')
        )
      }

父组件设置函数 `changeInput` ，然后再子组件处传入
子组件向父组件传值的原理是通过由父组件传递函数给子组件，在子组件运行父组件的形式，达到子向父组件传参

## 一个传参技巧
通过`{...this.props}` 这样的方式，可以一次性传递当前页面的所有由父组件传递过来的参数
比如有这样一个场景：
父组件 --> 子组件 --> 孙子组件
孙子组件要获取父组件传递的所有参数，就可以用这样形式获取，如果参数发生变化，也只需要改动父组件这一处

## 对于props的设定
### 设定传递参数的默认值，[name]是组件名字

    [name].defaultProps = {
        'username': 'yezi'
    }
    
### 设置props类型

    [name].propsTypes = {
        'username': React.PropTypes.string.isRequired
    }
设置 `username` 必须为 `string` 类型，`required` 设置不能为空

---

## 与VUE的父子传参方式对比

### vue父组件向子组件传参数的方式和react基本相似，都是通过传递props[name]的值
### vue子组件向父组件传参的方式和react有极大的不同
1. vue通过$emit(name, [props])函数达到向上传递，然后再父组件监听派发上来的函数，然后<font color='blue'>在父函数中处理</font>
2. react依然通过this.props方式向子组件传进父组件参数，<font color='blue'>在子组件中处理</font>





















  [1]: http://www.ruanyifeng.com/blog/2015/03/react.html