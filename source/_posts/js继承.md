---
title: js继承
date: 2017-07-31 16:22:50
tags: js
---

# js继承

------

## 类继承

### 基于类继承的步骤：
1. 用一个类的声明定义对象的结构
2. 实例化该类以创建一个新对象，每个创建的对象都有一套该类的所有实例属性的副本

### 类继承与[工厂模式][1]的对比： 
<font color='red'>工厂模式未能解决对象的类型，即由哪个构造函数创建的</font>

### 类继承的例子：

    // 创建父类
    var Person = function (name) {
        this.name = name
      }
      Person.prototype.getName = function () {
        return this.name
      }
    // 创建子类
    var Author = function (name, book) {
        Person.call(this, name)
        this.book = book
      }
      // 子类继承父类
      Author.prototype = new Person()
      Author.prototype.construtor = Author
      // 子类的公共方法只能放在继承后面
      
      // 这是改进方法
      // extend(Author, Person)
      
      Author.prototype.getBook = function () {
        return this.book
      }
      var yezi = new Author('yezi', '陪你度过漫长岁月')
      console.log(yezi.getName()) // yezi
      console.log(yezi.getBook()) // 陪你度过漫长岁月

### 改进代码： 

    // 定义继承方法
    // 利用空函数作为中转，因为空函数几乎不占内存，而且可以避免构造函数的副作用
    // 主要是new操作符会执行一遍父类函数，如果父类函数过大，则耗费大
      function extend (subClass, superClass) {
        var F = function () {}
        F.prototype = superClass.prototype
        subClass.prototype = new F()
        subClass.prototype.construtor = subClass
      }

### 类继承的对象bug

    // 重新定义父类
    var Person = function (name) {
        this.name = name
        this.behavior = {
          foot: true,
          eat: true
        }
      }
    // 其他代码同上
    // 创建两个实例
    var yezi = new Author('yezi', '陪你度过漫长岁月')
    var feifei = new Author('feifei', '岁月如歌')
    yezi.behavior = {
        foot: false,
        eat: false
    }
    console.log(yezi.behavior) // Object {foot: false, eat: false}
    console.log(feifei.behavior) // Object {foot: true, eat: true}

<font color='red'>总结：</font> 直接改变继承的对象值，相当于给实例添加新的属性behavior

接下来看看另外一个改变对象测试

    yezi.behavior.foot = false
    yezi.behavior.eat = false
    console.log(yezi.behavior) // Object {foot: false, eat: false}
    console.log(feifei.behavior) // Object {foot: true, eat: true}

<font color='red'>总结：</font> 通过这种方式，原理是实例会向上查找behavior属性，再找到foot和eat的值进行修改，而不是给实例直接添加新的behavior属性，这可以说是一个bug.

---


## 原型链继承

### 原型链继承区别之前提到的类继承，类继承沿用的是java面向对象的思维，而原型链则是利用js设计的思维
<font color='blue'>相对于类继承，原型链继承更加简单，但是依然避免不了引用类型的共享bug，原型链继承我理解为是浅复制，对于简单类型进行复制副本，引用类型则用‘指针’指向</font>

### 代码设计: 

    // 原型链继承的主要函数
    function clone (object) {
        var fn = function () {}
        fn.prototype = object
        return new fn()
    }

测试代码：

    var person = {
        name: 'default name',
        behavior: {
          foot: true,
          eat: true
        },
        hands: [1, 2, 3, 4, 5]
      }

      var author1 = clone(person)
      var author2 = clone(person)
      author1.name = 'yezi'
      author2.name = 'feifei'
      console.log('author1.name=', author1.name)
      console.log('author1.behavior=', author1.behavior)
      author2.behavior = {
        foot: false,
          eat: false
      }
      author2.hands.push(5)
      console.log('author1 hasownpropety', author1.hasOwnProperty('behavior'))
      console.log('author2 hasownpropety', author2.hasOwnProperty('behavior'))
      console.log('author2.name=', author2.name)
      console.log('author2.behavior=', author2.behavior)
      console.log('author1.hands=', author1.hands)
      console.log('author2.hands=', author2.hands)
      
### 结果：
![此处输入图片的描述][2]

### 分析： 
1. 利用 obj.hasOwnProperty(prop) 可以看出，将author2.behavior的值改变，<font color='red'>相当于给author2添加新的behavior属性，而author1自身不存在behavior属性</font>，author1通过原型链向上查找父类的behavior属性
2. 同时author1.hands的改变影响了author2.hands的值，所以这是一个浅复制


---


## new操作符的实现步骤
new共经历四个过程

    var fn = function () {}
    var fnObj = new fn()
    
1. 创建一个空对象
 
     var obj = new object()
    
2. 设置原型链

    obj._proto_ = fn.prototype
    
3. 让fn的this指向obj，并执行fn的函数体

    var result = fn.call(obj)

4. 判断fn的返回值类型，如果是值类型，返回obj。如果是引用类型，就返回这个引用类型的对象

    if (typeof(result) === 'object') {
      fnObj = result
    } else {
      fnObj = obj
    }
  
---

## 参考：

JavaScript设计模式 （2009年+(美国)(Rossharmes)哈梅斯(美国)(DustinDiaz)迪亚斯+《JavaScript设计模式》


  [1]: https://yezi12138.github.io/2017/05/11/%E5%B7%A5%E5%8E%82%E6%A8%A1%E5%BC%8FVS%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0/
  [2]: http://wx4.sinaimg.cn/mw690/a359ab18gy1fi354qbexwj207y06r0st.jpg