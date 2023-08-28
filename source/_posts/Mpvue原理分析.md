---
title: Mpvue原理分析
date: 2023-08-28 9:36:00
categories:
- hybird
tags:
- mpvue
- wechat
- mini-program
---

### 一、半编译半运行时

#### 1.1 【半编译】vue的template 通过 mpvue-template-compiler 编译成 wxml

![](https://img14.360buyimg.com/ling/jfs/t1/107613/2/2742/42346/5e09f5c5E7424d037/b2d7a1d5327cdf4f.jpg)

#### 1.2 【半运行时】在小程序中实现了一套runtime用来跑vue的script部分的代码

vue运行时

![](https://img14.360buyimg.com/ling/jfs/t1/98880/36/8962/23682/5e09ee76Ee70d5703/898f0c8b2003eafe.jpg)

mpvue运行时

![](https://img12.360buyimg.com/ling/jfs/t1/85977/36/9055/54025/5e09efc8Ea2a7e868/60360ca99a89c1a7.jpg)

&#x20;mpvue 的运行时，会首先将 patch 阶段的 DOM 操作相关方法置空，也就是什么都不做。其次，在创建 Vue 实例的同时，还会偷偷的调用 `Page()` 用于生成了小程序的 page 实例。然后 运行时的 patch 阶段会直接调用 `$updateDataToMp()`方法，这个方法会获取挂在在 page 实例上维护的数据 ，然后通过 `setData` 方法更新到视图层。

### 二、整体架构

![](/img/image_kidDls1l_H.png)

-   Vue实例与小程序Page建立关联
-   小程序和Vue生命周期建立映射关系，能在小程序生命周期中触发Vue生命周期
-   建立事件代理机制，在事件代理函数中触发与之对应的Vue事件响应
-   小程序与vue数据同步

### 三、整体思考

mpvue的局限性：

1.  由于小程序模版的限制，无法使用vue特有的api，如：filter、v-html
2.  将vue运行在小程序上，小程序和vue的运行时性能决定了mpvue的上限
3.  fork了vue\@2.4.1的代码，使得mpvue无法及时更新vue的新特性
