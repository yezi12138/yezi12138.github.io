---
title: 一个简单的MVVM(一)
date: 2017-08-14 14:41:15
tags: MVVM
---
# MVVM框架

------

## 前言：

基于 Vue 数据绑定方式实现MVVM框架

这次基于MVVM框架实现的使用方法：

    // 定义监听对象
    data = {
      a: "before",
      b: [1, 2, 3, 4],
      c: {
        d: "child d",
        f: "child f"
      }
    }
    var oberver = new Listen(data)
    
数据定义包括了基本类型，对象的嵌套，还有数组的处理
参考： [JavaScript实现MVVM][1]

## 实现原理：
通过 `Object.defineProperty` 方法，设置对象的 `setter` `getter` 构造器，完成对数据的监听

## 1. 对象的监听

定义监听函数 Listen：

    class Listen {
        constructor (obj) {
            // 对obj进行类型判断
            if (Object.prototype.toString.call(obj) !== '[object Object]') {
                return console.log('this obj isn't a object, please make sure')
            }
            this.observe(obj)
        }
        observe (obj) {
             // 遍历data里面的属性，设置set/getter
            Object.keys(obj).forEach((key, index, keyArray) => {
                var oldVal =  = obj[key]
                Object.defineProperty(obj, key, {
                    get: function () {
                        return oldVal
                    }.bind(this),
                    set: function (newVal) {
                        // 做一些处理，这里测试用了打印函数
                        console.log(`oldVal=${oldVal} --- newVal=${newVal}`)
                        oldVal = newVal
                    }.bind(this)
                })
            })
        }
    }
这里使用`Object.prototype.toString.call(obj)` 判断数据的类型，详细内容可以看这个： [JS数据类型判断][2]

### 测试代码：

    var data = {
      a: "before",
      b: [1, 2, 3, 4],
      c: {
        d: "child d",
        f: "child f"
      }
    }
    var observer = new Listen(data)
    data.a = "after"
    data.b = "after"
    
### 结果：
> oldVal=before --- newVal=after
> oldVal=1,2,3,4 --- newVal=after

好了，现在我们这个框架可以监听一级数据了，可是对于二级数据，比如 `data.c.d` ，修改它的值，却没有出现监听。
原因是代码里面只处理了一级数据，现在我们可以再加个判断，如果当前obj[key]是属于对象，则进行迭代

### 处理二级数据：

    if (Object.prototype.toString.call(obj[key]) === '[object Object]' || Object.prototype.toString.call(obj[key]) === '[object Array]') {
              this.observe(obj[key])
            }

将这段代码加到 ` Object.keys(obj).forEach` 下面，然后测试结果
> data.c.d = "after"
> data.b[1] = "after"

输出：
> oldVal=child d --- newVal=after
> oldVal=2 --- newVal=after

## 对数组方法的处理
原理： 在数组的原型链上加多一层，变成:
> objArray ---> addChain ---> Array

这样就可以监听Array方法，而且也不必重写Array的原生方法

### 具体实现：

    overrideArrayProto (array) {
          // 保存原始 Array 原型
          var originalProto = Array.prototype,
          // 通过 Object.create 方法创建一个对象，该对象的原型就是Array.prototype
          overrideProto = Object.create(Array.prototype),
          self = this,
          result;
          // 遍历要重写的数组方法
          var OAM = ['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse']
          Object.keys(OAM).forEach((key, index, keyArray) => {
            var method = OAM[index],
            oldArray = [];
            // 使用 Object.defineProperty 给 overrideProto 添加属性，属性的名称是对应的数组函数名，值是函数
            Object.defineProperty(overrideProto, method, {
              value: function () {
                // 复制数组
                // this指向array
                oldArray = this.slice(0)
                console.log('arguments: ', arguments)
                // 将arguments转化为数组
                var arg = [].slice.apply(arguments)
                result = originalProto[method].apply(this, arg)
                // 执行回调，这里测试用打印函数
                console.log(`oldArray=${oldArray} --- newVArray=${result}`)
                return result
              },
              enumerable: false
            })
          })
          // 最后 让该数组实例的 __proto__ 属性指向 假的原型 overrideProto
          array.__proto__ = overrideProto
        }

在`observe`开头加上一下代码：

    // 如果发现监测的对象是数组的话就要调用 overrideArrayProto 方法
          if(Object.prototype.toString.call(obj) === '[object Array]'){
            this.overrideArrayProto(obj)
          }

### 测试代码：

    data.b[1] = "after"
    data.b.pop()
    data.b.push(121)
    console.log(data.b)
    
### 结果输出：
> oldVal=2 --- newVal=after
> oldArray=1,after,3,4 --- newVArray=4
> oldArray=1,after,3 --- newVArray=4
> [1, "after", 3, 121]

一个简单的MVVM框架到此结束


  [1]: http://hcysun.me/2016/04/28/JavaScript%E5%AE%9E%E7%8E%B0MVVM%E4%B9%8B%E6%88%91%E5%B0%B1%E6%98%AF%E6%83%B3%E7%9B%91%E6%B5%8B%E4%B8%80%E4%B8%AA%E6%99%AE%E9%80%9A%E5%AF%B9%E8%B1%A1%E7%9A%84%E5%8F%98%E5%8C%96/
  [2]: http://www.cnblogs.com/flyjs/archive/2012/02/20/2360504.html99%AE%E9%80%9A%E5%AF%B9%E8%B1%A1%E7%9A%84%E5%8F%98%E5%8C%96/