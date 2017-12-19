---
title: npm publish发布插件
date: 2017-05-16 00:08:39
tags:
     -npm
     -vue
---


# 通过npm publish发布自己的插件

------

最近做一个项目需要用到轮播效果，于是就想着自己开发一个组件，可以通过npm install安装，这样可以循环利用我开发的组件

### 这是我的文件目录
![list](http://a3.qpic.cn/psb?/V11dGOEK4JH7xZ/joJSPWVIJmxRBbz2fnAd9m3IWIW88J0o4qpCVX.tMsI!/b/dG0BAAAAAAAA&bo=wgCZAAAAAAADB3k!&rf=viewer_4)



### 第一步是创建入口文件，这里我举例创建swiper.js文件

    import swiper from '../src/swiper'; //这是我所需要的组件
    export default  swiper;   //将组件导出


------

### 在根目录下进入cmd，输入npm init 生成package.json文件

其中 ：
 name  这是通过npm install的文件名
 version 这是版本号，每次发布的版本号不能相同
 description 这是对插件的描述
 repository 这是github的仓库
 
 下面是我创建的package文件代码
 

    {
      "name": "banner-swiper",
      "version": "1.0.3",
      "description": "simple swiper for vue",
      "main": "dist/swiper.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "repository": {
        "type": "git",
        "url": "git+ssh://git@github.com/yezi12138/swiper.git"
      },
      "author": "yezi12138",
      "license": "ISC",
      "bugs": {
        "url": "https://github.com/yezi12138/swiper/issues"
      },
      "homepage": "https://github.com/yezi12138/swiper#readme"
    }


---

### 接下来就到了发布环节

首先登陆npm账号（没有就去注册一个）
> npm login

会提示你输入名字、密码和邮箱

然后发布
> npm publish

如果发布出错，错误是you don't have permission...可能是你的name已经被创建了，换个名字再试试



贴上我插件的github地址：https://github.com/yezi12138/swiper