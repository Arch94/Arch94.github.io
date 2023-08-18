---
title: ionic篇学习笔记-部署项目编
date: 2018-03-18 13:35:00
categories:
- ionic
tags:
- 部署
---
##1.启动ionic项目
首先是随便都能查到的npm安装命令
```
npm install -g ionic cordova
ionic start myIonicProject blank
cd myIonicProject
ionic serve
```
看一下项目目录，了解一下
首先src里存放的ionic的angular代码
其次www 里存放的是src里的typescript代码编译后的注入的js代码，当然还有其他的东西，比如说图片，sass编译后注入的css代码，都是用webpack打包编译生成。
www很重要，就算ionic打包android安装包也是把www移过去。
##2.配置移动开发环境
其中ios的开发环境如果是苹果电脑就不用配了，ios只能在苹果上开发。
android的环境配置网上一大堆，就是安装sdk配下环境变量。如果想要贪便宜，可以直接下个androidStudio，神器啊，快速帮助新手配置环境。
##3.打包android安装包
首先先给ionic项目添加支持的平台，在项目目录下运行命令行
```
ionic cordova platform
```
能看到一大串支持的平台列表(android、ios、browser)和已经安装的平台
运行命令安装ionic的android平台
```
ionic cordova platform add android
```
添加完以后，可以看到目录下的platforms文件夹下多了个android文件夹，里面就是android项目代码，然后再运行命令build一下android平台的代码。
```
ionic cordova build android
```
这行命令行就是为了更新android平台下的代码，其中就有把www替换android项目里的www。

在mac下build完以后，最好进到上一层目录, sudo chmod -R 777 myIonicProject
不然可能androidStudio没权限打开，windows下也应该用管理员权限设置一下。

值得一提的是，这个build很多人都会出现问题，如果是android环境没配好，确保你的androidStudio能新建一个项目并成功跑起来。然后如果对android开发不太熟，不知道gradle怎么用怎么打包的筒子（作者我～～）,直接用androidStudio打开ionic编译后生成的android项目，就是那个platform下的android文件夹。androidStudio会自动帮你构建项目。

我在androidStudio编译项目时遇到了一些问题，如果编译失败，除了网速问题（科学上网）以外，还遇到过
![Jietu20171218-134359@2x.png](https://upload-images.jianshu.io/upload_images/6114493-80d4abe9ba203c9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这种问题，现在ionic build项目我已经没遇见了，前几个月build一次遇见一次。
直接去 https://www.imooc.com/article/21867 看，找到问题6就行了。

android项目导入androidStudio没问题以后，直接运行，就能看到自己的app跑起来了。

ionic项目开发起来很方便，但是真正难的就是他的底层环境配置。也不能说他坑，因为他是基于cordova开发的，cordova配置环境本来就很坑。打个cordova的jar包用官方的命令根本打不起来，遇上了好多问题，最后在androidStudio的帮助下才成功。

