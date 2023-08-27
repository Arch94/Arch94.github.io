---
title: express手写
date: 2023-08-27 15:11:00
categories:
- Express
tags:
- Express
---

#### 执行代码

```javascript
let express = require('./index.js')
let app = express();
// 参数不写就是 /
app.use(function(req,res,next){
  console.log('middleware')
  res.setHeader('Content-Type','text/html;charset=utf-8')
  next('name error')
})

app.get('/name',function(req,res){
  res.end('name')
})

app.all('*',function(req,res){
  res.end('all')
})
app.use(function(err,req,res,next){
  console.log(err)
  next(err)
})
app.use(function(err,req,res,next){
  console.log(err+' 2')
  next()
})
app.post('/age',function(req,res){
  res.end('age')
})
app.listen(3001,function(){
  console.log('server start 3001')
})

```

#### 手写代码

```javascript
let http = require('http');
let url = require('url')
function createApplication(){
  // app 是一个监听函数
  let app = (req,res)=>{
    // 取出每一个layer
    // 1.获取请求的方法
    let m = req.method.toLowerCase();
    let { pathname } = url.parse(req.url,true)
    let index = 0;
    // 递归
    function next(err){
      // 数组全部迭代完毕还没有找到，说明路径不存在
      if(index === app.routes.length){
        return res.end(`Cannot find ${m} ${pathname}`)
      }
      let {method, path, handler} = app.routes[index++]
      if(err){
        // 如果有错误 我们应该去找错误中间件，错误中间件有四个参数
        if(handler.length === 4){
          handler(err,req,res,next)
        } else {
          // 如果没有匹配到，要将error继续传递下去
          next(err)
        }
      } else {
        if(method === 'middle'){
          if(path === '/' || path === pathname || pathname.startsWith(path+'/')){
            handler(req,res,next);
          } else {
            next(); // 如果这个中间件没有匹配到，继续走下一个
          }
        } else {
          if(
            (method === m || method === 'all') && 
            (path === pathname || path === '*')
          ){
            handler(req,res)
          } else {
            next();
          }
        }
      }
      
    }
    next();
  }
  // 路径是一个数组
  app.routes = [];
  app.use = function(path,handler){
    if(typeof handler !== 'function'){
      handler = path;
      path ='/'
    }
    let layer = {
      method:'middle',
      path,
      handler
    }
    app.routes.push(layer)
  }
  app.all = function(path,handler){
    let layer = {
      method:'all',
      path,
      handler
    }
    app.routes.push(layer)
  }
  http.METHODS.forEach(method=>{
    method = method.toLocaleLowerCase();
    app[method] = function(path,handler){
      let layer = {
        method,
        path,
        handler
      }
      app.routes.push(layer)
    }
  })
  app.listen = function(){
    let server = http.createServer(app)
    server.listen(...arguments)
  }
  return app;
}
module.exports = createApplication

```

#### express 基本规则

&#x20;1\.  路由匹配(`app.get`、`app.post` ...)，`app.all` 全路由匹配
&#x20;2\.  `app.use(path,handler)` 中间件匹配，只有`handler`话`path`默认为` /`
&#x20;3\.  中间件用next执行下一个中间件，next一旦传入信息则会去寻找错误中间件
&#x20;   错误中间件有4个参数

#### express 总结

1.  express中间件和路由按顺序放入routes，
2.  当执行中间件的时候，会传递next，使得下一个中间件或者路由得以执行
3.  当执行到路由的时候就不会传递next，也使得routes的遍历提前结束
4.  当执行完错误中间件后，会继续执行后面的中间件和路由
