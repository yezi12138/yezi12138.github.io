---
title: node上传文件
date: 2017-06-11 21:46:52
tags:  --node
---

# node入门——http上传文件服务器

------

这是第一次学习nodeJS这么语言，参考 Manuel Kiessling 大神的教学文案，结合自己的理解做一个应用允许用户浏览页面以及上传文件，我这里做了一点分析和解答

用到的知识有：

> * http模块
> * 路由知识
> * formidable模块

目录结构如下：
+node学习
    &nbsp; |  --index.js
    &nbsp; |  --requestHandles.js
    &nbsp; |  --router.js
    &nbsp; |  --server.js

---

### 入口文件提供接口，server用来启动服务器，而路由是用来确定进入的页面，并且传递数据，handles是用来返回数据给客户端。

#### 入口文件index.js

    var server = require("./server");
    var router = require("./router");
    var requestHandlers = require("./requestHandlers");
    
    var handle ={}
    handle["/"]= requestHandlers.start;
    handle["/start"]= requestHandlers.start;
    handle["/upload"]= requestHandlers.upload;
    
    server.start(router.route, handle);
    
    

#### server.js

    var http = require("http");
    //url用来解析请求的URL路径
    var url = require("url");
    
    function start(route, handle){
      function onRequest(request, response){
        var postData =""; //用来保存上传的数据
        var pathname = url.parse(request.url).pathname; //路径名字
        console.log("Request for "+ pathname +" received.");

    request.setEncoding("utf8");

    request.addListener("data",function(postDataChunk){
      postData += postDataChunk;
      console.log("Received POST data chunk '"+
      postDataChunk +"'.");
    });

    request.addListener("end",function(){
    //request请求全部结束后执行，转向对应的路由，执行相应函数
      route(handle, pathname, response, postData);
    });

     }
    //启动服务
      http.createServer(onRequest).listen(8888);
      console.log("Server has started.");
    }
    
    exports.start = start;
    
    
    
#### router.js

      function route(handle, pathname, response, postData){
      console.log("About to route a request for "+ pathname);
      if(typeof handle[pathname]==='function'){
        handle[pathname](response, postData);
      }else{
        console.log("No request handler found for "+ pathname);
        response.writeHead(404,{"Content-Type":"text/plain"});
        response.write("404 Not found");
        response.end();
      }
    }
    
    exports.route = route;
    
    
#### requesthandles.js

    var querystring = require("querystring");
    
    function start(response, postData){
      console.log("Request handler 'start' was called.");
    
      var body ='<html>'+
        '<head>'+
        '<meta http-equiv="Content-Type" content="text/html; '+
        'charset=UTF-8" />'+
        '</head>'+
        '<body>'+
        '<form action="/upload" method="post">'+
        '<textarea name="text" rows="20" cols="60"></textarea>'+
        '<input type="submit" value="Submit text" />'+
        '</form>'+
        '</body>'+
        '</html>';
    
        response.writeHead(200,{"Content-Type":"text/html"});
        response.write(body);
        response.end();
    }
    
    function upload(response, postData){
      console.log("Request handler 'upload' was called.");
      response.writeHead(200,{"Content-Type":"text/html"});
      //确保中文不会乱码
      response.write('<head><meta charset="utf-8"/></head>'); 
      response.write("You've sent the text: "+
      querystring.parse(postData).text);
      response.end();
    }
    
    exports.start = start;
    exports.upload = upload;
    
<font color="red">这样子我们就实现了一个简单的文本输入，显示的小应用，接下来继续完善这个应用</font>

--- 

## 处理文件上传

首先要引入一个模块
 > npm install formidable
 
官方的使用介绍如下：

        var formidable = require('formidable'),
        http = require('http'),
        util = require('util');
    
    http.createServer(function(req, res){
      if(req.url =='/upload'&& req.method.toLowerCase()=='post'){
        // parse a file upload
        
        //创建Formidable.IncomingForm对象
        var form =new formidable.IncomingForm();
        form.parse(req,function(err, fields, files){
          res.writeHead(200,{'content-type':'text/plain'});
          res.write('received upload:\n\n');
          res.end(util.inspect({fields: fields, files: files}));
        });
        return;
      }
    
      // show a file upload form
      res.writeHead(200,{'content-type':'text/html'});
      res.end(
        '<form action="/upload" enctype="multipart/form-data" '+
        'method="post">'+
        '<input type="text" name="title"><br>'+
        '<input type="file" name="upload" multiple="multiple"><br>'+
        '<input type="submit" value="Upload">'+
        '</form>'
      );
    }).listen(8888);
    
#### Q：什么是util
> util是node.js核心模块，提供常用函数的集合，用于弥补核心js的功能


#### Q：util.inspect
util.inspect(object,[showHidden],[depth],[colors])是一个将任意对象转换为字符串的方法，
通常用于调试和错误的输出。
1. showhidden - 是一个可选参数，如果值为true，将会输出更多隐藏信息。
2. depth - 表示最大递归的层数。如果对象很复杂，可以指定层数控制输出信息的多少。
3. 如果不指定depth,默认会递归3层，指定为null表示不限递归层数完整遍历对象。
4. 如果color = true，输出格式将会以ansi颜色编码，通常用于在终端显示更漂亮的效果。
<font color="red">注意：</font>特别要指出的是，util.inspect并不会简单地直接把对象转换为字符串，即使对该对象定义了toString方法也不会调用

#### Q：form的enctype="multipart/form-data"？
是设置表单的MIME编码。默认情况，这个编码格式是application/x-www-form-urlencoded，不能用于文件上传；只有使用了multipart/form-data，才能完整的传递文件数据


### 上传代码解析

        function upload(response, request){
      console.log("Request handler 'upload' was called.");
    
      var form =new formidable.IncomingForm();
      console.log("about to parse");
      form.uploadDir="./public";//必须设置
      form.parse(request,function(error, fields, files){
        console.log("parsing done");
        console.log(files);
        fs.renameSync(files.upload.path,"./public/1.jpg");
        response.writeHead(200,{"Content-Type":"text/html"});
        response.write("received image:<br/>");
        response.write(`<img src='/show' />`);
        response.end();
      });
    }
    
//设置上传的目录位置
> form.uploadDir="./public";

//更改文件的名字
> fs.renameSync(files.upload.path,"./public/1.jpg") 

//这段无法显示图片
> response.write(`<img src='./public/1.jpg' />`);

//如果直接写相对路径或者绝对路径无法显示，原因不明（Q,Q）
> response.write(`<img src='/show' />`);

--- 

接下来就按照教程继续下去就行，我就不累赘（http://ourjs.com/detail/529ca5950cb6498814000005）