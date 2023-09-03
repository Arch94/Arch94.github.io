---
title: vue-router实现
date: 2023-09-03 13:55:00
categories:
- vue
tags:
- vue
---
# vue-router实现

SPA（single page）需要在**不刷新页面**的情况下做页面更新的能力，SPA前端路由是利用了浏览器的hash或history属性。

#### hash和history

1.  hash 路由：监听 url 中 hash 的变化，然后渲染不同的内容，这种路由不向服务器发送请求，不需要服务端的支持；
    -   hash （url中#后面的部分）虽然出现在 URL 中，但不会被包含在 http 请求中，对后端完全没有影响，因此**改变 hash 不会重新加载页面**
    -   **当hash改变时，会触发hashchange事件**，监听该事件，对页面进行更新。
    -   hash值改变会改变浏览器的历史记录
    > hash 模式下，仅 # 之前的内容包含在 http 请求中，对后端来说，即使没有对路由做到全面覆盖，也不会报 404
2.  history 路由：监听 url 中的路径变化，需要客户端和服务端共同的支持；
    > 所以vue用history 模式时，一般需要在nginx 对路径进行处理下，指向index.html文件
    -   history 利用了 html5 history interface 中新增的\*\* pushState() 和 replaceState() **方法，这**两个方法应用于浏览器记录栈 \*\*。
    -   window\.onpopstate()监听url改变
    -   原地刷新时⇒后台指向固定地点
        在当前已有的 back、forward、go 基础之上，它们提供了对历史记录 修改的功能（pushState将传入url直接压入历史记录栈，replaceState将传入url替换当前历史记录栈）。

#### vue-router插件实现

1.  实现一个静态install方法，因为作为vue插件都必须有这个方法，给Vue.use()去调用；并通过mixin挂载\$router实例
    ```javascript
    KVueRouter.install = function(_vue){
        Vue = _vue;
        //全局混入
        Vue.mixin({
            beforeCreate(){//拿到router的示例，挂载到vue的原型上
                if (this.$options.router) {
                    Vue.prototype.$router=this.$options.router;
                    this.$options.router.init();
                }
            }
        })
    }
    ```
2.  监听路由变化（hashChange）；
3.  解析配置的路由，即解析router的配置项routes，能根据路由匹配到对应组件；
4.  实现两个全局组件router-link和router-view；（最终落地点）
    ```javascript
    Vue.component('router-view',{
        render:(h)=>{
          //这里使用了data中的 this.app.current，根据vue相应式原理，当变化 this.app.current时
          //这里会监听并发生变化
            const Component = this.$routerMap[this.app.current].component;
            return h(Component)
        }
    });
    ```
