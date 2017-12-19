---
title: js封装
date: 2017-07-29 15:57:25
tags: js
---
# js的封装

------

## 用一个简单的book设计来做例子

### 1. 门户大开型对象

#### <font color='blue'>特点：</font>

 >1. 成员属性公开
 >2. 用this指向成员属性
 >3. 外部可改变成员值，不利于维护
 
#### <font color='blue'>代码设计：</font>

    var Book = function (isbn, name, author) {
      this.isbn = isbn
      this.name = name
      this.author = author
    }
    // 公共方法
    Book.prototype.display = function () {
      // ...
    }
 

##### <font color='red'>总结：</font> 这是最简单的设计模式，不过保密性差

#### <font color='blue'>代码改进：</font>

    var Book = function (isbn, name, author) {
        this.setIsbn(isbn)
        this.setName(name)
        this.setAuthor(author)
      }  
      // 可以在赋值器中对数据进行处理
      Book.prototype = {
        display: function () {
        },
        setIsbn: function (isbn) {
          this.isbn = isbn
        },
        getIsbn: function () {
          return this.isbn
        },
        setName: function (name) {
          this.name = name
        },
        getName: function () {
          return this.name
        },
        setAuthor: function (author) {
          this.author =  author
        },
        getAuthor: function () {
          return this.author
        }
      }
      var book = new Book('123456', '西游记', '陈乔恩')
      // 下面是测试构造器
      console.log(book.constructor === Book) // false
      // 因为用prototype = {} 这种方式会断开constructor
      Book.prototype.constructor = Book
      console.log(book.constructor === Book) // true
      
##### <font color='red'>总结：</font> 改进之后降低了代码直接的耦合度，同时优化了代码，便于维护

--- 

### 1. 私用成员设计模式

#### <font color='blue'>特点：</font>

 >1. 拥有私有成员
 >2. 只能通过特权方法修改私有属性
 >3. 使用了闭包来设计
 >4. <font color='red'>私有成员不用this指向</font>
 >5. <font color='red'>特权方法用this指向</font>
 
#### <font color='blue'>代码设计：</font>

    var Book = function (isbn, name, author) {
      // 私有属性
      var isbn =isbn , name = name, author = author
      // 特权方法，用来访问私有属性
      this.setIsbn = function (newIsbn) {
        // 对数据处理
      }
      this.getIsbn = function () {
        return isbn
      }
      this.getName = function () {
        return name
      }
      this.getAuthor = function () {
        return author
      }
    }
    Book.prototype.display = function () {}
    
##### <font color='red'>总结：</font> 私有属性只能通过特权方法访问，私有属性不同this指向，特权方法要用this指向.每个实例都有一个副本，各不影响

---

### 1. 静态成员设计模式

#### <font color='blue'>特点：</font>

 >1. 拥有静态成员
 >2. 静态成员是作用在类上，不是实例上面
 >3. 通过立即调用函数和闭包方式创建

#### <font color='blue'>代码设计：</font>

    var Book = (function () {
      // 私人静态成员
      var bookNum = 0
      // 私人静态方法
      var total = function () {}
      
      // 返回constructor
      return function (isbn, name, author) {
      // 私有属性
      var isbn =isbn , name = name, author = author
      // 特权方法，用来访问私有属性
      this.setIsbn = function (newIsbn) {
        // 对数据处理
      }
      this.getIsbn = function () {
        return isbn
      }
      this.getName = function () {
        return name
      }
      this.getAuthor = function () {
        return author
      }
      this.getBookNum = function () {
        return bookNum
      }
      bookNum++  // 静态成员值加一
    } 
    })();
    
    // 静态方法 (即每个实例都有一个副本)
    Book.borrow = function () {}
    
    // 公共方法 (所有实例共享的方法)
    Book.prototype.display = function () {}
    
这个方法中 `bookNum` 是所有实例共享的数据

### 常量

#### 可以用静态成员方法模仿，只设置取值器



### 参考：

#### JavaScript设计模式 （2009年+(美国)(Rossharmes)哈梅斯(美国)(DustinDiaz)迪亚斯+《JavaScript设计模式》