---
title: 用hexo开发博客教程
date: 2017-05-10 12:37:58
tags:
---
# 用hexo开发博客教程

------

这篇文章主要是介绍如何运用hexo部署个人博客，并且将博客上传到github，更换next主题，并且关联leancloud和百度注册。

> * 配置环境
> * 安装hexo，开发第一个博客
> * 将博客上传到github
> * 更换next主题


------

## 配置环境

本次开发主要环境如下：
> * window10系统 
> * node.js       [安装node.js教程链接](https://nodejs.org/en/)
> * git  [安装git教程链接](https://git-scm.com/book/zh/v1/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)
> * 注册github账号 [github官网](https://github.com/github)



------

## 安装hexo，开发第一个博客

### 1. github创建仓库

登陆github，点击右上角+号，点击new repository;
![Alt text](http://wx1.sinaimg.cn/thumb150/00617eZdgy1ffgaro9w08j308f06874a.jpg)

**注意：**这里的Repository Name一定要是你的  <font color=red><用户名>.github.io</font>
![Alt text](http://wx3.sinaimg.cn/thumb150/00617eZdgy1ffgarslskbj30qn0gpdhh.jpg)

---

### 2. 配置github仓库公钥
打开git界面，第一次登陆的话要设定用户
git config --global user.name “yourname"
git config --global user.email "yourEmail"



比如：git config --global user.name "yezi12138"
git config --global user.email "xxxxxxx@qq.com"

**生成SSH公钥：** ssh-keygen -t rsa -C "your_email@youremail.com" 
**查看SSH公钥：**  cat ~/.ssh/id_rsa.pub


点击settings，Deploy keys  设置好公钥
![Alt text](http://wx2.sinaimg.cn/thumb150/00617eZdgy1ffgarv2zxkj30sj0fhwgm.jpg)




---

## 3.安装hexo，开发第一个博客

在自己认为合适的地方创建一个文件夹，这里我以E：/text 为例子讲解
打开电脑的命令行工具  （FN+R 输入cmd） 
**切换到目录下**
```seq
cd /d/text
```
![Alt text](http://wx4.sinaimg.cn/thumb150/00617eZdgy1ffgarpn8nfj30r305mdfu.jpg)


**安装Hexo**

```seq
npm install hexo-cli -g
```
```seq
npm install hexo --save
```
**初始化Hexo，代码带$符号的表示在git界面操作，其余在cmd中操作**
```seq
$ hexo init
```
**安装依赖**
```seq
npm install
```
**部署和发布**
```seq
$ hexo g
```
```seq
$ hexo d
```
然后会提示：
>    INFO Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.

在浏览器中打开http://localhost:4000/，你将会看到：
![Alt text](http://wx1.sinaimg.cn/thumb150/00617eZdgy1ffgas3fjxij311y0lcqk5.jpg)

到目前为止，Hexo在本地的配置已经全都结束了.

---


## 将博客上传到github

**同样在根目录下找到_config.yml文件中，找到Deployment，然后按照如下修改：**

> type: git
 > repo: git@github.com:gdutxiaoxu/gdutxiaoxu.github.io.git
>  branch: master

<font color=red>repo里面的地址是你仓库的SSH：</font>
![Alt text](http://wx1.sinaimg.cn/thumb150/00617eZdgy1ffgartppimj30sh05ewf4.jpg)

**注意：**type 、repo这些属性后面有一个空格

**重新部署发布**
```seq
$ hexo g
```
```seq
$ hexo d
```
**踩坑注意点1**：如果出现以下提示：
> ERROR Deployer not found: git

则需要在文件目录下安装deployer：
> npm install hexo-deployer-git --save

**踩坑注意点2**：如果出现以下提示：
> Permission denied (publickey). 
fatal: Could not read from remote repository. 
Please make sure you have the correct access rights 
and the repository exists.

则是因为没有设置好public key所致，需要重新设置好key


成功之后
![Alt text](http://wx1.sinaimg.cn/thumb150/00617eZdgy1ffgartueggj30ew04pq3n.jpg)


**输入：**https://yourName.github.io  就可以看到你部署的博客了

![Alt text](http://wx3.sinaimg.cn/thumb150/00617eZdgy1ffgasariw1j311i0jlq5k.jpg)

---

## 更换next主题

**next使用文档：**http://theme-next.iissnan.com/

里面有详细的更换教程，我就不重复说明了，主要讲一下可能踩到的坑

**注意：**

> * 从官网clone的next主题已经是配置好的了，只需要按教程一步一步往下走就行了
> * 关联阅读次数统计（LeanCloud）的class名一定要是：Counter （C是大写的）


---
本文参考了gdutxiaoxu的博客，网址：http://blog.csdn.net/gdutxiaoxu/article/details/53576018
