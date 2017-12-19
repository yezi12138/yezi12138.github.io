---
title: ES6 promise原理
date: 2017-08-06 18:41:08
tags: ES6
---
# ES6 Promise原理

------

最近在学爬虫，对继步的操作十分烦恼，因为需要用到异步函数。于是开始研究起异步编程，现在看到ES6的promise异步，觉得十分有趣，于是上网查了查它的原理。

参考： mengera88的文章
地址链接：[mengera88的文章][1]

## 简单的Promise函数

    function promise (fn) {
        // 定义状态，一共有三种 ‘pending’， ‘fulfilled’，‘rejected’ 
        var state = 'pending',
        // 定义回调组，因为可能存在多个函数
        callbacks = [],
        // 定义在resolve（value）传进去的值
        value = null
        this.then = function (onFulfilled) {
            // ...
        }
        function resolve (newValue) {
            // ...
        }
        fn(resolve)
      }
      
一个简单的promise框架已经搭设好，这里没有加入<font color='red'>rejected</font>回调，这个以后再说

### 测试promise用的函数
现在我们再看看怎么调用promise

    function getUserId() {
        return new promise (function(resolve) {
          //模拟异步请求
          setTimeout(function () {
            resolve('success')
          }, 2000)
        })
      }
      // 主函数
      getUserId().then(function (value) {
        console.log(value)
      })
      

 1. 调用then方法，将想要在Promise异步操作成功时执行的回调放入callbacks队列，其实也就是注册回调函数，可以向观察者模式方向思考；
 2. 创建Promise实例时传入的函数会被赋予一个函数类型的参数，即<font color='red'>resolve</font>，它接收一个参数<font color='red'>value</font>，代表异步操作返回的结果，当一步操作执行成功后，用户会调用resolve方法，这时候其实真正执行的操作是将callbacks队列中的回调一一执行。

### 改进promise  （完善then和resolve）

    function promise (fn) {
        var state = 'pending',
        callbacks = [],
        value = null
        this.then = function (onFulfilled) {
          // 在等待时间保存传进来的函数 
          if (state === 'pending') {
            callbacks.push(onFulfilled)
            // return this 是为能链式调用then
            return this
          }
          // 保证在异步结束后，后续函数能正常执行
          onFulfilled(value)
          return this
        }
        // 防止resolve在then完成之前执行
        function resolve (newValue) {
          value = newValue
          state = 'fulfilled'
          // 将resolve置于队列尾部，防止在then之前就执行
          setTimeout(function () {
            callbacks.forEach(function (callback) {
              callback(value)
            })
          }, 0)
        }
        fn(resolve)
      }

这个样子我们再运行上面测试promise的代码，结果就是异步调用成功

## 链式promise

上面的代码虽然可以实现异步，但却无法实现链式异步，就是连续两个promise。假设有这样一个场景：你get请求一个数据，然后之后的post方法依赖get过来的请求，这就需要到达链式异步

### 测试用的代码

    function fn1() {
        return new promise(function (resolve) {
          setTimeout(function () {
            console.log('fn1 is over')
            resolve('success')
          }, 3000)
        })
      }
      function fn2(val) {
        console.log('fn2 receipt:', val);
        return new promise(function (resolve) {
          setTimeout(function () {
            console.log('fn2 is over')
            resolve('fali')
          }, 5000)
        })
      }
      // 主函数
      fn1().then(fn2).then(function (val) {
        console.log(val)
      })

### 1. 对then方法修改

    this.then = function (onFulfilled) {
          return new promise(function (resolve) {
            handle({
              onFulfilled: onFulfilled,
              resolve: resolve
            })
          })
        }
这是then返回一个promise对象，handle的数据保存在promise1中的callbacks

### 2. 对resolve修改

    function resolve (newValue) {
        // 这是将fn2 return回的promise对象进行判断，如果是函数或者对象，就执行
        // 如果不太明白可以跳过，我下面会描述过程
          if (newValue && (typeof newValue == 'object' || typeof newValue === 'function')) {
            var then = newValue.then
            if (typeof then === 'function') {
              then.call(newValue, resolve)
              return
            }
          }
          value = newValue
          state = 'fulfilled'
          // 将resolve置于队列尾部，防止在then之前就执行
          setTimeout(function () {
            callbacks.forEach(function (callback) {
              handle(callback)
            })
          }, 0)
        }


### 全部代码

    window.onload = function () {
      function promise (fn) {
        var state = 'pending',
        callbacks = [],
        value = null
        this.then = function (onFulfilled) {
          return new promise(function (resolve) {
            handle({
              onFulfilled: onFulfilled,
              resolve: resolve
            })
          })
        }
        function handle (callback) {
          if (state === 'pending') {
            callbacks.push(callback)
            return
          } else if (state === 'fulfilled') {
            var ret = callback.onFulfilled(value) || value
            callback.resolve(ret)
          }
        }
        // 防止resolve在then完成之前执行
        function resolve (newValue) {
          if (newValue && (typeof newValue == 'object' || typeof newValue === 'function')) {
            var then = newValue.then
            if (typeof then === 'function') {
              then.call(newValue, resolve)
              return
            }
          }
          value = newValue
          state = 'fulfilled'
          // 将resolve置于队列尾部，防止在then之前就执行
          setTimeout(function () {
            callbacks.forEach(function (callback) {
              handle(callback)
            })
          }, 0)
        }
        fn(resolve)
      }

      function fn1() {
        return new promise(function (resolve) {
          setTimeout(function () {
            console.log('fn1 is over')
            resolve('success')
          }, 3000)
        })
      }
      function fn2(val) {
        console.log('fn2 receipt:', val);
        return new promise(function (resolve) {
          setTimeout(function () {
            console.log('fn2 is over')
            resolve('fali')
          }, 5000)
        })
      }
      // 主函数
      fn1().then(fn2).then(function (val) {
        console.log(val)
      })
    }

<font color='red'>函数执行过程：</font>

 1. 第一步执行<font color='blue'>fn1函数</font>,创建promise1，这时因为计时器异步，所以直接执行then方法，把<font color='blue'>fn2函数</font>添加到promise1的callbacks上，同时，因为then同时返回了一个promise2对象，所以<font color='blue'>fn3函数</font>添加到promise2上
 2. 执行fn1计时器中的函数，然后执行promise1的resolve，传入'success'，因为传入的字符串，resolve执行handle(callback)，执行<font color='blue'>if (state === 'fulfilled')里面的语句</font>。这个时候被执行的是<font color='blue'>fn2函数</font>
 3. var ret = callback.onFulfilled(value) || value 执行后首先执行fn2里面的输出语句，然后将返回的promise3对象传给ret
 4. 执行resolve(ret)，就是走到resolve函数的if判断语句，<font color='red'>then.call(newValue, resolve)结果等同于执行new promise4(resolve)</font>
 5. 因为创建promise4,同理会先执行同步函数then，将fn3链接到promise4的callbacks，然后等promise4的计时器执行完毕，执行fn3
 6. 总的指向为： promise1 --> fn2 &&  fn2 return的promise4 --> fn3

## 总结
是不是觉得很绕？ 绕就对了。。。。慢慢看多几次写几个console.log就会慢慢慢慢慢慢慢慢理解。


  [1]: https://segmentfault.com/a/1190000009478377
