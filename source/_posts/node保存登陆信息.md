---
title: node保存登陆信息
date: 2017-06-17 20:18:58
tags: -node
---

# node+vue实现记住密码功能

------

首先要引入依赖

    var session = require('express-session');
    var cookieParser = require('cookie-parser');
    var MongoStore = require('connect-mongo')(session);//用来将session添加到mongodb数据库，实现长时间保存
    
使用session（在build\dev-server.js）
   

     app.use(cookieParser());
        app.use(session({
          secret:'secret',
          resave:true, // don't save session if unmodified
          saveUninitialized:false,// don't create session until something stored
          cookie:{
            maxAge:1000*60*10 //过期时间设置(单位毫秒)
          },
          store: new MongoStore({   //创建新的mongodb数据库
                host: 'localhost',    //数据库的地址，本机的话就是127.0.0.1，也可以是网络主机
                port: 27017,          //数据库的端口号
                db: 'test'        //数据库的名称。
              })
        }));


### 这样子的代码有一个报错：
> Error: Connection strategy not found MongoDB

解决方法是store里面要有url，不知道是不是版本问题：

    store: new MongoStore({   //创建新的mongodb数据库
        url: 'mongodb://localhost/test' 
      })

### 再次有一个报错：
>  Cannot set property 'user' of undefined

百度了一下原因是要把sesson的配置放在路由之前。

    app.use(cookieParser());
    app.use(session({
      ~
    }));
    app.use(api) //使用路由
    
### 增加一个路由，用来返回session的值

  
    router.get('/auth',(req,res) => {
        if(req.session.user){
            res.send(req.session.user)
        }
    });
    
#### 在index主页下增加方法，用来获取用户信息

    /*************************************************
    获取登录信息
    @ 允许自动登录则返回登陆状态码 1
    @ 不允许则返回登陆状态码 2
    **************************************************/
    router.get('/auth',(req,res) => {
        let token = {
            account: req.session.user,
            loginCode:req.session.loginCode
    }
    if(token){
        res.send(token)
    }
    });
      
#### 添加自动登录的函数

    autoLogin(){
        this.$http.get('/auth').then((res) => {
          // console.log(res.data)
          var token = res.data;
          if(token && token.loginCode === 1){
            this.account = token.account;
            this.isLogin = true;
          }else{
            this.account = token.account;
            this.isLogin = false;
          }
        })
      }
      
#### 在页面创建时候判断是否自动登录

    created(){
      this.autoLogin();
    }


这样就可以在页面创建时候验证用户信息，实现免登录。

