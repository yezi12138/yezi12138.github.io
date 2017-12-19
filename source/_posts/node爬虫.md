---
title: node爬虫
date: 2017-07-20 20:32:25
tags:  -node
---
# node爬虫

------

为了获取豆瓣书籍的数据，上网学习了怎么用node写一个爬虫，爬豆瓣网站信息

## 准备工作

先新建一个文件夹 起名 node爬虫
然后进入cmd，执行
> npm init // 初始化项目

安装对应依赖
> npm install express cheerio superagent --save

新建一个主文件 index.js
项目结构如下
![pic2][1]

--- 

## 编写index.js

    var express = require('express')
    var cheerio = require('cheerio')
    var superagent = require('superagent')
    
    var app = express()
    
    app.get('/', function (req, res, next) {
      superagent.get('https://book.douban.com/')
      .end(function (err, sres) {
        if (err) {
          return next(err)
        }
        // 这里的$和jq操作类似
        var $ = cheerio.load(sres.text)
        var books = getData('.books-express', $)
        // 可以把上面代码注释，把下面注释去掉，获取不同数据
        // var books = getData('.popular-books', $)
        res.send(JSON.stringify(books))
      });
    });
    app.listen(3000, function () {
      console.log('app is listening at port 3000');
    });
    
主要函数

    function getData (className,$) {
      var books = {}
      $(className).each(function (idx, element) {
        var $el = cheerio.load(element)
        var section = []
        var book = {}
        var lis = $el('ul li').each(function(i, elem) {
            // 这里实验所以只取前6个数据
          if (i>=6) {
            return
          }
          book = {
            img: $(this).find('.cover a img').attr('src'),
            title: $(this).find('a').attr('title'),
            rating: $(this).find('.info .average-rating').text().replace(/[\r\n]/g, ""),
            author: $(this).find('.info .author:first-child').text().replace(/[\r\n]/g, ""),
            classification: $(this).find('.info .book-list-classification').text().replace(/[\r\n]/g, ""),
            reviews: $(this).find('.info .reviews').text().replace(/[\r\n]/g, "").split('(')[0],
            reviewer: $(this).find('.info .reviews a').text().replace(/[\r\n]/g, "")
          }
          section.push(book)
        })
        books[$(this).find('h2 span:first-child').text()] = section
        })
        return books
    }
    
### <font color='red'>注意：</font>必须要用$(this).find把内容限定在每一个li内部检索，不然会检索到其他部分内容，造成内容重叠或者缺失
### 同时用replace(/[\r\n]/g, "")是为了去除文本中的\n换行符

---

## 最后获取到的数据如下

> {"新书速递":[{"img":"https://img3.doubanio.com/lpic/s29478140.jpg","title":"不疯魔不成活","rating":"","author":" 微笑的猫 ","classification":"","reviews":"","reviewer":""},{"img":"https://img1.doubanio.com/lpic/s29471397.jpg","title":"六神磊磊读唐诗","rating":"","author":" 六神磊磊 ","classification":"","reviews":"","reviewer":""},{"img":"https://img3.doubanio.com/lpic/s29460644.jpg","title":"隐性逻辑","rating":"","author":" [德] 卡尔·诺顿 ","classification":"","reviews":"","reviewer":""},{"img":"https://img3.doubanio.com/lpic/s29498174.jpg","title":"权力脸谱","rating":"","author":" 铲史官 ","classification":"","reviews":"","reviewer":""},{"img":"https://img1.doubanio.com/lpic/s29498208.jpg","title":"父母：挑战","rating":"","author":" [美] 鲁道夫•德雷克斯 ","classification":"","reviews":"","reviewer":""},{"img":"https://img3.doubanio.com/lpic/s29490136.jpg","title":"π的杀人魔法","rating":"","author":" 墨殇 ","classification":"","reviews":"","reviewer":""}]}

---

## 把数据写入文件
引入新的包： 
>  var fs = require('fs')

添加新的函数：

    function writeFile(file, data){  
    // appendFile，如果文件不存在，会自动创建新文件  
    // 如果用writeFile，那么会删除旧文件，直接写新文件  
    fs.appendFile(file, data, function(err){
        if (err) {
          console.log("fail " + err)
        } else {
          console.log("写入文件ok")
        }
    })
}

更新主函数：

    app.get('/', function (req, res, next) {
      superagent.get('https://book.douban.com/')
      .end(function (err, sres) {
        if (err) {
          return next(err)
        }
        var $ = cheerio.load(sres.text)
        var books = getData('.books-express', $)
        // var books = getData('.popular-books', $)
        res.send(JSON.stringify(books))
        
        // 添加如下代码
        writeFile('C:\\Users\\Administrator\\Desktop\\node爬虫\\data.txt', JSON.stringify(books))
      });
    });
    
这样就可以把数据写入本地文件
![pic1][2]
![pic3][3]


  [1]: http://wx4.sinaimg.cn/mw690/a359ab18gy1fhqmh7md4jj205p030jra.jpg
  [2]: http://wx1.sinaimg.cn/mw690/a359ab18gy1fhqmh7lnrmj20fz04tglz.jpg
  [3]: http://wx2.sinaimg.cn/mw690/a359ab18gy1fhqmh7ink6j207m055dfu.jpg