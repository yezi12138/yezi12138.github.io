---
title: 工厂模式VS构造函数
date: 2017-05-11 14:27:46
tags:
     - js
     - 学习笔记
---
# 工厂模式VS构造函数

------

### 什么是工厂模式？
简单来说，如果我们要使用同一个接口创建很多个对象，我们就会想到用函数传递参数的方法：

    function createPerson(name,age,job){
        var o = new Object();
        o.name = name;
        o.age = age;
        o.job = job;
        o.sayName = function(){
            alert(this.name);
        };
        return o;
    }
    var person1 = createPerson("yezi",22,"web");
    var person2 = createPerson("join",39,"designer");

 这种方式虽然可以无数次调用，却没办法解决对象识别的问题（即怎样知道一个对象的类型）。随着js发展，有一个新的模式可以解决这个问题。
 
 
### 构造函数

使用构造函数重写上面的例子：

    function Person(name,age,job){
        this.name = name;
        this.age = age;
        this.job = job;
        this.sayName = function(){
            alert(this.name);
        };
    }
    var person1 = new Person("yezi",22,"web");
    var person2 = new Person("join",39,"designer");

在这个例子中我们注意到一下几个不同点：

 - 没有显示创建对象  //var o = new Object()
 - 直接将属性和方法赋给了this对象
 - 没有return语句
 
要创建Person的新实例，必须要使用new操作符，以这种方式调用构造函数实际上会经历一下4个步骤：
 1. 创建一个新对象
 2. 将构造函数的作用于给新对象（因此this指向了这个新对象）
 3. 执行构造函数的代码
 4. 返回新对象
 
总的来说构造函数隐形地创建了新对象，并且会继承一个属性 **constructor**

我们可以用constructor来判断实例的类型

    alert（person1.constructor == Person） //true
    alert（person2.constructor == Person） //true
    
    
### 将构造函数当作函数

构造函数和其他函数唯一区别，就是调用方式不同。不过，构造函数毕竟也是函数。任何情况，只要通过new操作符来调用，那它就可以作为构造函数；而任何函数，如果不通过new操作符来调用，那它和别的普通函数也不会有什么不同。例如之前的例子可以这样调用：

    //当作构造函数使用
    var person1 = new Person("yezi",22,"web");
    person.sayName（）;//"yezi"
    
    //当作普通函数使用
    Person("yezi",22,"web"); //添加到window对象
    window.sayName（）;//"yezi"
    
    //在另一个对象的作用域中使用
    var o = new Object();
    Person.call(o,"yezi",22,"web");
    o.sayName();//"yezi"
    
    
### 顺便一提:
#### 使用构造函数创建对象时候，如果函数内部有多个相同属性（上面例子的sayName函数），则可能会出项内存占多的情况。因为函数本质也是对象，在创建实例的时候也同时在内部创建不同对象（sayName）

    alert（person1.sayName == person2.sayName） //false

对于多个实例共享的函数，建议把函数写在类型的原型链中

    function Person(name,age,job){
        this.name = name;
        this.age = age;
        this.job = job;
        Person.prototype.sayName = function(){
            alert(this.name);
        };
    }
    
    alert（person1.sayName == person2.sayName） //true
    
    