---
title: mongodb安装和使用
date: 2017-06-13 17:22:27
tags:  --mongodb
---
# mongodb使用

------

## 1.安装mongodb（https://www.mongodb.com/download-center#community）

## 2.配置环境变量，让mongodb可以全局使用
在【我的电脑】-->属性-->高级设置-->环境变量 path下设置mongodb下bin的路径
> C:\Program Files\MongoDB\Server\3.4\bin

这样就可以在任意cmd下启动mogodb

## 3.启动mongodb
> mongod --dbpath=D:\data\db //dbpath是指定数据存放目录

成功的话会输出一下内容：
2017-04-09T10:45:31.583+0800 I CONTROL  [initandlisten] MongoDB starting : pid=12244 port=27017 dbpath=E:\Tool\modb\db 64-bit host=JSB015
2017-04-09T10:45:31.584+0800 I CONTROL  [initandlisten] targetMinOS: Windows Vista/Windows Server 2008
...


## 4.一个简单的mongodb操作实例
    
     var mongoose = require("mongoose");
    mongoose.Promise = global.Promise; 
    var db = mongoose.connect("mongodb://127.0.0.1:27017/test");
    var TestSchema = new mongoose.Schema({
        name : {type:String},
        age : {type:Number,default:0},
        email : {type:String},
        time : {type:Date,default:Date.now}
    });
    var TestModel = db.model("test1",TestSchema); 
    var TestEntity = new TestModel({
        name:'helloworld',
        age:28,
        emial:'helloworld@qq.com'
    });
    TestEntity.save(function(err,doc){
        if(err){
            console.log("error :" + err);
        } else {
            console.log(doc);
        }
    });
    
Schema ： 一种以文件形式存储的数据库模型骨架，不具备数据库的操作能力

Model ： 由Schema发布生成的模型，具有抽象属性和行为的数据库操作对

Entity ： 由Model创建的实体，他的操作也会影响数据库

## 5.踩坑：

 1. 

     DeprecationWarning: Mongoose: mpromise (mongoose's default promise library) is deprecated, plug in your own promise library instead: http://mongoosejs.com/docs/promises.html
如果出现这段代码，则需要在db.connect之前加入以下代码：
> mongoose.Promise = global.Promise;
//或者 这样 ：mongoose.Promise = require('bluebird');

 2. 
 

    MongoError: failed to connect to server [127.0.0.1:27017] on first connect
这是mongodb数据库没有开启提示的错误，应该先开启数据库连接
> mongod
// 或者指定数据目录 mongod --dbpath=d:\data\db

## 6.可以让mongodb随window启动而启动
这样就不用每次都启动一次
在命令行执行：
> mongod --logpath D:\MongoDb\logs\MongoDB.log --logappend --dbpath D:\data --directoryperdb --serviceName MongoDB --install --auth

删除自启动服务：
>  sc delete MongoDB