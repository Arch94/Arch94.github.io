---
title: 记一次hexo 博客上线 经验总结
date: 2018-12-03 00:09:47
categories:
- hexo
tags:
- pm2
- nginx
- hexo
---

前两天买了一台百度云服务器花了11块钱，花了点时间学了下如何部署hexo博客以及一些服务器方面的配置，没用hexo推荐的方式部署。专门记一个文章。系统用的是centos，先上图和链接
## 博客地址
[sunqx的博客](http://www.sunqx.top:4000)

![preview.png](https://upload-images.jianshu.io/upload_images/6114493-3d1caea56828ef4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 步骤
1. 先用hexo快捷搭建本地blog，再把项目上传到github 
2. 本地ssh无密码登陆远程服务器，把服务器的防火墙打开并配置指定端口才可以登陆
3. 远程服务器配置 ssh key 后再把项目从git clone到云服务器
4. 再用pm2，直接本地快捷部署项目到服务器并重启
5. 使用nginx 转发端口 

## 配置要点

#### 先用hexo快捷搭建本地blog
首先解释下
hexo 基于nodejs的快速、简洁且高效的博客框架，它是帮人快速搭建个人博客的。
PM2是node进程管理工具,可以利用它来简化很多node应用管理的繁琐任务,如性能监控、自动重启、负载均衡等,而且使用非常简单。就是一开始上手感觉好容易报错。

这里只将一些官网上没有的，如何本地跑起来一个hexo 项目和部署一个主题请看官网 [hexo官网]( https://hexo.io/zh-cn/)

`` hexo server ``  是项目 的启动命令 ，默认hexo启动端口是4000。但这样启动项目，nodejs是一个单线程项目，万一某一次程序出错，那整个服务就挂了。所以需要pm2 来监控这项线程，万一出错pm2还能自动重启。

pm2 启动命令一般是`` pm2 start app.js``  那如何用pm2 来管理hexo呢，可以通过在项目目录下新建一个 app.js
```sh
var spawn = require('child_process').spawn;
free = spawn('hexo', ['server', '-p 4000']);/* 其实就是等于执行hexo server -p 4000*/

free.stdout.on('data', function (data) {
    console.log('hexo standard output:\n' + data);
});

free.stderr.on('data', function (data) {
    console.log('hexo standard error output:\n' + data);
});

free.on('exit', function (code, signal) {
    console.log('hexo child process eixt ,exit:' + code);
});
```
如此一来在项目根目录下  就能用 ``pm2 start app.js`` 来跑hexo项目了

#### 本地ssh无密码登陆远程服务器，服务器 ssh key 配置
初次配置的云服务器需要安装这些工具包
```
 sudo yum install vim openssl build-essential libssl-dev wget curl git
 yum install gcc-c++
```
本地配置ssh key
1. 本地 cd /
2. cd .ssh  或者 open ~/.ssh  
3. ls
    ``id_rsa  id_rsa.pub  known_hosts``
4. 备份
   `` mv id_rsa id_rsa_backup``
   ``mv id_rsa.pub id_rsa_backup.pub``
5. 生成ssh公钥
``ssh-keygen -t rsa -b 4096 -C "931035063@qq.com" ``
6. 查看
``cat id_rsa``
``cat id_rsa.pub``
7. key作用生效
``eval "$(ssh-agent -s)"``
``ssh-add ~/.ssh/id_rsa``
8. 然后在连上**远程服务器** ``cd ~/.ssh`` ,编辑服务器端授权文件 ``vi authorized_keys``
9. 在在**客户端** 输入 ``cat id_rsa.pub``将带有邮箱的公钥复制进去，再:wq!退出authorized_keys
10.  配置生效 ``sudo service sshd restart`` 
11.  然后就可以直接通过 ssh root@xx.xx.xx.xx 来免密码 登陆了

注意点:
- 因为我连上云服务器后也没创建新用户，直接用root 编辑的，其他人配置是要注意一下 权限问题，我看网上也有不少人因为权限问题而生效失败
- 如果还是配置没生效，需要看一下  [sshd_config配置详解](https://www.cnblogs.com/jingwu/articles/5598340.html)
RSAAuthentication yes
PubkeyAuthentication yes
这两项不能为no
- 还有一个地方，每次ssh登陆输ip太过于麻烦，当然你用secureCRT这样的工具当我没说, **在mac下面 可以配置快捷连接云服务器的命令**
-- 在``sudo vim ~/.bashrc``
-- 新加 ``alias ssh_baiduyun="ssh root@xx.xx.xx.xx"``
-- ``source .bashrc`` 以后即可通过 ``ssh_baiduyun`` 连接服务器
- 再重申一点 本地的 ``id_rsa.pub`` 对应的是 服务器上的 ``authorized_keys``,两个复制错了就尴尬了
#### 在服务器配置ssh并clone git 项目
生成服务器端公钥 和本地一样
``cd ~/.ssh``
``ssh-keygen -t rsa -b 4096 -C "931035063@qq.com"``
``eval "$(ssh-agent -s)"``
``ssh-add ~/.ssh/id_rsa``
生效,centos 下``service sshd restart``
把  ``id_rsa.pub`` 里面的内容 放到git 账户里面ssh 配置里
![Screenshot 2018-12-02_23-25-37.png](https://upload-images.jianshu.io/upload_images/6114493-9b765e356cd0a732.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**注意！！** 在服务器里你准备clone 项目的文件夹里 先用ssh clone 一遍 ``git@github.com:SHpoi/my-blog.git`` 第一次git clone 会有确认项，不然后面用pm2部署老是会遇到一个拉取项目为空的报错。

#### 在服务器配置nodejs环境
服务器配置nodejs 用的是nvm，安装nvm
```
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
```
**nvm 安装以后重新连接云服务器**，不然环境变量不会生效
nvm安装nodejs，依次执行以下命令
```
nvm install v8.9.0
nvm use v8.9.0
nvm alias default v8.9.0
npm --registry=https://registry.npm.taobao.org install -g npm
```
拉取完以后记得先试用``hexo server``着能不能跑起来,进入文件夹先``npm install``再看还差什么其他的全局依赖

#### 在本地使用pm2 一键部署云服务器上的项目
这一块真的是血泪啊，就2行命令行，报的错查了一下午。
1. 在本地项目目录下新建一个 ecosystem.json，与app.js平级
```
{
  "apps" : [{
    "name": "myApp",
    "script": "app.js",
    "instances": 1,
    "autorestart": true,
    "env": {
      "NODE_ENV": "development"
    },
    "env_production": {
      "NODE_ENV": "production"
    }
  }],
  "deploy" : {
    "production" : {
      "user" : "root",
      "host" : "xx.xx.xx.xx",
      "port" : "22",
      "ssh_options": "StrictHostKeyChecking=no",
      "ref"  : "origin/master",
      "repo" : "git@github.com:SHpoi/my-blog.git",
      "path" : "/root/www/myblog/",
      "env": {
        "NODE_ENV": "production"
      },
      "pre-setup": "rm -rf /var/www/myblog/production/source",
      "post-deploy" : "npm install && pm2 startOrRestart ecosystem.json --env production",
      "env"  : {
        "NODE_ENV": "production"
      }
    }
  }
}

```
然后运行2行命令 第一行在云服务器搭建运行环境，第二行每次重新发布项目需要
```
pm2 deploy ecosystem.json production setup
注意有巨坑，第一次ssh clone 会校验，要先在该目录下clone 一遍git ssh 连接，之后才会成功。不然会一直报空链接
pm2 deploy ecosystem.json production --force
force一定要加不然git会报冲突不解决
```
以后每次发布博客
``hexo new  '记一次hexo 博客上线 经验总结'`` 生成md文件，写完以后提交到git仓库
再用命令行在本地项目执行下 ``pm2 deploy ecosystem.json production --force`` 即可

##### pm2 配置注意点
- ``pm2 deploy ecosystem.json production --force`` force一定要加不然git会报冲突不解决
-  运行setup命令时 第一次ssh clone 会校验，要先在该目录下clone 一遍git ssh 连接，之后才会成功。不然会一直报空链接。第一次clone完删了即可。
- 启动前先删了source里面的git 代码，不然pm2 会报冲突不解决而部署失败
``` 
"pre-setup": "rm -rf /var/www/myblog/production/source",
- 部署后运行 命令进行项目依赖包安装 再跑起来hexo项目 
"post-deploy" : "npm install && pm2 startOrRestart ecosystem.json --env production",
```
-  ``"script": "app.js"`` 为了让json 找到app.js,所以放平级，这个json文件如果不是私有仓库不建议提交，当然我是提了，不过你们有高手千万别看了我的git项目的配置文件就攻击它，活动买的加上域名服务就12块钱。
- ``/root/www/myblog/`` 我是为了方便我登进去就能找项目才放root下的。还有这个目录需先创建，不然pm2不知道部署在哪。

- ``pm2 deploy ecosystem.json  production setup `` 有时候还需要再次执行，因为有时候会出现这种错误 ``bash: line 0: cd: /root/www/myblog/source: 没有那个文件或目录``

#### nginx配置
这一块没啥说的 无非就是端口转发， 80转4000
nginx 安装
```
sudo yum install nginx
```
**安装完重连服务器，不然怎么输命令配置都不生效**

```
upstream myblog{
 server localhost:4000;
}

server {
    listen  80 ;
    server_name  www.sunqx.top;

    location / {
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;
          proxy_set_header Host $http_host;
          proxy_set_header X-Nginx-Proxy true;
          proxy_pass http://myblog;
          proxy_redirect off;
    }
}
```
nginx 配置生效
```
sudo nginx -t 
sudo nginx -s reload
```











