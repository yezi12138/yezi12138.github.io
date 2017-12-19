---
title: 一个简单的MVVM(二)
date: 2017-09-01 20:41:39
tags: MVVM
---
# 一个简单的MVVM（二）

------

一个MVVM框架有五个部分，分别是：
1. `sf.js` 框架的入口，定义了框架的类
2. `watcher.js` 用来监听data的数据改变，原理是赋值器劫持
3. `scanner.js` 扫描dom树，找出`sf-` 标志的属性和`{{value}}` 的写法，并且生成map
4. `render.js` 用来渲染对应属性数据
5. `boundItem.js` 定义scanner的map的类型和相关`sf-` 监听函数

这次是仿写VUE的MVVM框架，用法和VUE基本类似

## 用法：

    var sf = new SegmentFault({
      data: {
        message1: '单向绑定',
        message2: 'this is message2',
        value: '双向绑定',
        arr: {
          name: 'yezi',
          song: ['五月天', '知足', '倔强'],
          love:{
            live:{
              a:'yezi 3 time'
            }
          }
        }
      }
    })
    
`data` 用来绑定数据
`SegmentFault` MVVM框架的类


## SegmentFault 类

    let SegmentFault = class SegmentFault {
      constructor ({data = {}} = {}) {
        this.data = data
        // 用来维护viewModel和被绑定的view之间的关系
        this.viewViewModelMap = []
        this.init()
      }
      // 类的初始化
      init () {}
      // 手动更新dom
      refresh () {}
      
      
## watcher.js

    export class Watcher {
      constructor(sf) {
        this.sf = sf
        this.data = sf.data;
        // 监听数据，然后执行回调,这里是更新dom树
        this.observe(sf.refresh)
      }
      observe(callback) {
        let data = this.data;
        let sf = this.sf;
        // 定义循环函数，监听多层对象，详细可以看一个简单的MVVM（一）
        var define = function (data) {
          Object.keys(data).forEach((key) => {
            let value = data[key]
            Object.defineProperty(data, key, {
              get: function () {
                return value
              },
              set: function (newValue) {
                if (newValue === value) {
                  return
                }
                value = newValue
                callback.call(sf)
              }
            })
            if (Object.prototype.toString.call(data[key]) === '[object Object]' || Object.prototype.toString.call(data[key]) === '[object Array]') {
              define(data[key])
            }
          })
        }
        define(data)
      }
    }
    
    
## scanner.js 

    import {BoundItem} from './boundItem'
    export class Scanner {
      constructor (sf) {
        this.sf = sf
        this.results = []
      }
      // 遍历dom下面所有节点，寻找sf- 和{{}}
      findSf (dom) {
        let children = dom.children
        // 判断children是否为纯数组
        if (!children.push) {
          children = [].slice.call(children)
        } 
        children.forEach((key, index) => {
          let children = key.children
          if(children && children.length > 0) {
            // 如果子节点还有节点，则进行递归
            this.findSf(key)
          } else {
          // 获取节点所有属性
            var atrs = key.attributes
            atrs = [].slice.call(atrs)
            // 如果在该节点找到{{}}型数据，则bindData为true，否则为false
            let bindData = /\{\{(\w){1,}([\.\w]*)(\w){1,}\}\}/g.test(key.innerText)
            let type = null , expression = null, el = key
            if (atrs.length !== 0) {
            // 遍历属性，找到sf-标签的属性，保存属性名和值
              atrs.forEach((atr) => {
                if (/^(sf-)/g.test(atr.name)) {
                  type = atr.name
                  expression = atr.nodeValue
                }
              })
            }
            // 保存数据，形成dom --> 数据的map
            let item = new BoundItem(type, expression, el, this.sf, bindData)
            this.results.push(item)
          }
        })
      }
      scanBindDOM () {
        let body = document.getElementsByTagName('body')[0]
        this.findSf(body)
        return this.results
      }
    }
    
    
    
## boundItem.js

    export class BoundItem {
      constructor(type = null, expression = null, element = null, sf = null, bindData = null) {
        this.type = type;
        this.expression = expression;
        this.el = element;
        this.bindData = bindData;
        this.sf = sf;
        this.listen()
      } 
      // 这里只用两个属性做例子
      listen () {
        let el = this.el
        let sf = this.sf
        let expression = this.expression
        if (el.nodeName === 'INPUT') {
          el.addEventListener('input', () => {
            sf.data[expression] = el.value
          }, false)
        }
      }
    }
    
    
## render.js

    export class Renderer{
      constructor (data) {
        this.data = data
      }
      render (boundItem) {
        let objValue = null
        let {type, expression, el, sf, bindData} = boundItem
        // 替换{{}}
        var pattern1 = new RegExp(/\{\{(\w){1,}([\.\w]*)(\w){1,}\}\}/g)
        // 绑定sf-
        var pattern2 = new RegExp(`/\{\{${type}\}\}/g`)
        // bindData为true表示存在{{}}
        if (bindData) {
          // 保存{{message}}
          let expressionText = el.innerText.match(pattern1)[0]
          // 取得message，使得可以从data中获取真实数据
          let expressionData  = expressionText.match(/(\w){1,}([\.\w]*)(\w){1,}/g)[0]
          // 多重对象绑定的取值 比如 arr.name.song
          let objArr = expressionData.split('.')
          objValue = this.data[objArr[0]]
          console.log('objArr:', objArr)
          // 循环取值
          if (objArr.length >= 2) {
            objArr.forEach((obj, index) => {
              if (index >= 1) {
                objValue = objValue[obj]
              }
            })
          }
          // 替换模板，赋值
          el.innerText = el.innerText.replace(expressionText, objValue)
          // 这一步是为了防止不断渲染
          boundItem.bindData = false
        }
        // 对特定类型的处理
        if (type === 'sf-text') {
          el.innerHTML = this.data[expression]
        } else if (type === 'sf-value') {
          el.value = this.data[expression]
        }
      }
    }
    
    
## 最后，再重写主函数

    import {Watcher} from './watcher';
    import {Scanner} from './scanner';
    import {Renderer} from './render';
    let SegmentFault = class SegmentFault {
      constructor ({data = {}} = {}) {
        this.data = data
        // 用来维护viewModel和被绑定的view之间的关系
        this.viewViewModelMap = []
        this.init()
      }
      init () {
        // 监听sf-数据变化
        let watcher = new Watcher(this)
        let scanner = new Scanner(this)
        this.render = new Renderer(this.data)
        this.viewViewModelMap = scanner.scanBindDOM()
        this.viewViewModelMap.forEach((boundItem) => {
          this.render.render(boundItem)
        })
      }
      refresh () {
        this.viewViewModelMap.forEach((boundItem) => {
          this.render.render(boundItem)
        })
      }
    }
    // 测试代码
    var sf = new SegmentFault({
      data: {
        message1: '单向绑定',
        message2: 'this is message2',
        value: '双向绑定',
        arr: {
          name: 'yezi',
          song: ['五月天', '知足', '倔强'],
          love:{
            live:{
              a:'yezi 3 time'
            }
          }
        }
      }
    })
    
    setTimeout(() => {
      console.log('setTimeout')
      sf.data.arr.name = 'feifei'
      // sf.data.value = 'change message2'
    }, 3000);
    
## 结尾
实现了：
1. {{message}}的文本绑定
2. {{arr.love.live}}实现多层对象绑定
3. 双向绑定sf-value
4. 单向绑定sf-text
5. data数据监听

未实现：
1. 同一个节点多个{{message}}绑定（在render加一层循环即可实现）
2. 其他功能