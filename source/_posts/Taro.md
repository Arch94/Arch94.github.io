---
title: Taro技术分析
date: 2023-08-29 10:00:00
categories:
- hybird
tags:
- Taro
- wechat
- mini-program
---

## 目录

-   [Taro 是什么？](#Taro-是什么)
-   [Taro2](#Taro2)
    -   [Taro2编译问题](#Taro2编译问题)
    -   [Taro2的架构特点](#Taro2的架构特点)
    -   [Taro2的架构问题](#Taro2的架构问题)
-   [Taro3](#Taro3)
    -   [Taro3的taro-runtime，自行实现了一套 DOM/BOM的api](#Taro3的taro-runtime自行实现了一套-DOMBOM的api)
    -   [Taro3的react适配](#Taro3的react适配)
    -   [Taro的react适配具体实现](#Taro的react适配具体实现)
    -   [如何将Taro的DOM tree渲染到小程序的视图层？](#如何将Taro的DOM-tree渲染到小程序的视图层)
    -   [Taro3 小程序Vue 实现](#Taro3-小程序Vue-实现)
    -   [Taro3 事件机制](#Taro3-事件机制)
    -   [Taro3的架构特点](#Taro3的架构特点)
    -   [Taro3的架构优缺点自我分析](#Taro3的架构优缺点自我分析)
    -   [Taro3的运行时优化](#Taro3的运行时优化)
        -   [精简的 DOM/BOM API](#精简的-DOMBOM-API)
        -   [Taro3更新优化](#Taro3更新优化)
        -   [包体积优化](#包体积优化)

### Taro 是什么？

> 📌多端统一开发解决方案

> 使用Taro，只书写一套代码，再通过Taro的编译工具，将源代码分别编译出可以在不同端（微信小程序、H5、RN等）运行的代码。

### Taro2

> 📌Taro2 是一个重编译时轻运行时的框架

Taro2 架构主要分为：**编译时** 和 **运行时**。其中编译时主要是将 Taro 代码通过 [Babel](https://babeljs.io/ "Babel") 转换成 小程序的代码，如：`JS`、`WXML`、`WXSS`、`JSON`。运行时主要是进行一些：生命周期、事件、data 等部分的处理和对接（运行时和React没有关系的）

![](/img/image_LbpYX-WYMf.png)

#### Taro2编译问题

![](/img/image_ecTM4qsics.png)

静态template转动态JSX相对简单，但是反过来却相当困难，这是因为JSX过于灵活，Taro2采用穷举的方式对JSX写法进行一一适配，工作量大。

#### Taro2的架构特点

-   **重编译时，轻运行时**：这从两边代码行数的对比就可见一斑。
-   编译后代码与 React 无关：Taro 只是在开发时遵循了 React 的语法，但与React DSL 强绑定
-   直接使用 Babel 进行编译：这也导致当前 Taro 在工程化和插件方面的羸弱。

#### Taro2的架构问题

-   React DSL 强绑定
-   React新特性需要手动对接
-   JSX适配工作量大，限制多
-   错误栈复杂，没有sourceMap（和React没关系，错误是从内部抛出的）

## Taro3

> 📌Taro3重运行时跨端框架

#### **Taro3的taro-runtime，自行实现了一套 DOM/BOM的api**

无论开发这是用的是什么框架，React 也好，Vue 也罢，最终代码经过运行之后都是调用了浏览器的那几个 BOM/DOM 的 API ，如：`createElement`、`appendChild`、`removeChild` 等。

![](https://img10.360buyimg.com/ling/jfs/t1/91591/19/8966/30891/5e09f712E6e7f1df8/c3cbbead85810461.jpg)

因此，我们创建了 [taro-runtime](https://github.com/NervJS/taro/tree/next/packages/taro-runtime "taro-runtime") 的包，然后在这个包中实现了 **一套 高效、精简版的 DOM/BOM API**（下面的 UML 图只是反映了几个主要的类的结构和关系）：

![](https://img12.360buyimg.com/ling/jfs/t1/107806/8/2825/294525/5e09f810E8d32975f/ec84acddae669131.png)

然后，我们通过 Webpack 的 [ProvidePlugin](https://webpack.js.org/plugins/provide-plugin/ "ProvidePlugin") 插件，注入到小程序的逻辑层。

#### Taro3的react适配

![](https://img10.360buyimg.com/ling/jfs/t1/93547/36/9145/40000/5e09f948Edd646b65/ce37c0d099b7f702.jpg)

最上层是 React 的核心部分 `react-core` ，中间是 `react-reconciler`，其的职责是维护 `VirtualDOM` 树，内部实现了 `Diff/Fiber` 算法，决定什么时候更新、以及要更新什么。

而 `Renderer` 负责具体平台的渲染工作，它会提供宿主组件、处理事件等等。例如 `React-DOM` 就是一个渲染器，负责 DOM 节点的渲染和 DOM 事件处理。

**Taro的React的适配，本质上是实现了一个React的自定义的Renderer，通过 taro-react包来连接 react-reconciler 和 taro-runtime**

![](https://img13.360buyimg.com/ling/jfs/t1/90851/28/9324/88846/5e0d9374Ec9b1a530/6a494163f77c6dd7.jpg)

#### Taro的react适配具体实现

具体的实现主要分为两步：

1.  实现 `react-reconciler` 的 `hostConfig` 配置，即在 `hostConfig` 的方法中调用对应的 Taro BOM/DOM 的 API。
2.  实现 render 函数（类似于 `ReactDOM.render`）方法，可以看成是创建 `Taro DOM Tree` 的容器。

![](https://img14.360buyimg.com/ling/jfs/t1/93510/31/8909/48959/5e09fa87E1d831f7e/5d6f1f945c23e529.jpg)

#### 如何将Taro的DOM tree渲染到小程序的视图层？

> 📌小程序组件模版化，Taro Dom Tree 递归渲染模版

我们将小程序的所有组件挨个进行**模版化处理**，从而得到小程序组件对应的template模版，然后基于template将Taro Dom Tree 进行递归渲染

![](https://img10.360buyimg.com/ling/jfs/t1/91642/11/9016/45574/5e09fc9dE03db038c/f87d7fc69e04b26e.jpg)

整个 Taro Next 的 React 实现流程图如下：

![](https://img11.360buyimg.com/ling/jfs/t1/106748/9/9067/61495/5e09fd1eE47733747/a11b3ef8b1a7f791.jpg)

#### Taro3 小程序Vue 实现

别看 React 和 Vue 在开发时区别那么大，其实在实现了 BOM/DOM API 之后，它们之间的区别就很小了。

Vue 和 React 最大的区别就在于运行时的 `CreateVuePage` 方法，这个方法里进行了一些运行时的处理，比如：生命周期的对齐。

![](https://img30.360buyimg.com/ling/jfs/t1/96356/5/8961/15030/5e0aae8aE452804bd/f4c7df38129d99c5.jpg)

其他的部分，如通过 BOM/DOM 方法构建、修改 DOM Tree 及渲染原理，都是和 React 一致的。

#### Taro3 事件机制

首先的 Taro Next 事件，具体的实现方式如下：

1.  在 小程序组件的模版化过程中，将所有事件方法全部指定为 调用 ev 函数，如：`bindtap`、`bindchange`、`bindsubmit`等。
2.  在 运行时实现 `eventHandler` 函数，和 eh 方法绑定，收集所有的小程序事件
3.  通过 `document.getElementById()` 方法获取触发事件对应的 `TaroNode`
4.  通过 `createEvent()` 创建符合规范的 `TaroEvent`
5.  调用 `TaroNode.dispatchEvent` 重新触发事件

![](https://img10.360buyimg.com/ling/jfs/t1/103912/33/8987/52864/5e0ab09fEcdcb31c2/d29510c9bc5dd255.jpg)

可以看到，Taro Next 事件本质上是**基于 Taro DOM 实现了一套自己的事件机制**，这样做的好处之一是，无论小程序是否支持事件的冒泡与捕获，Taro 都能支持。

#### Taro3的架构特点

-   **无 DSL 限制**：无论是你们团队是 React 还是 Vue 技术栈，都能够使用 Taro 开发
-   **模版动态构建**：和之前模版通过编译生成的不同，Taro Next 的模版是固定的，然后基于组件的 template，动态 “递归” 渲染整棵 Taro DOM 树。
-   **新特性无缝支持**：由于 Taro Next 本质上是将 React/Vue 运行在小程序上，因此，各种新特性也就无缝支持了。
-   **社区贡献更简单**：错误栈将和 React/Vue 一致，团队只需要维护核心的 taro-runtime。
-   **基于 Webpack**：Taro Next 基于 Webpack 实现了多端的工程化，提供了插件功能。

#### Taro3的架构优缺点自我分析

-   优点
    -   无DSL限制
    -   新特性无缝支持，因为Taro3本质是将Vue/React运行在小程序上，因此，各种新特性也可以无缝支持（Taro只是定义了一套DOM/BOM api，vue/react 核心代码依然正常运行）
-   缺点
    -   由于taro3的编译方案（多一层Taro Dom/Bom作中转，而且本质是将react/vue运行在小程序），性能上肯定不及Taro2/uniapp等静态模版编译方案
    -   Taro3的template递归渲染方式存在一定缺陷
        -   初始化全量setData模版渲染，因此初始化性能比较低
        -   视图更新速度取决于更新幅度，一旦出现大面积的渲染更新，setData数据量会比静态模版渲染方式高，性能也会随之降低

### Taro3的运行时优化

前面提到，同等条件下，编译时做的工作越多，也就意味着运行时做的工作越少，性能会更好。

![](https://img20.360buyimg.com/ling/jfs/t1/85061/33/9155/42050/5e0ae1afE263741c9/7a1664f1a6769d7a.jpg)

可以发现，相比原生小程序，Taro Next 多了红色部分的带来的性能隐患，如：引入 React/Vue 带来的 包的 Size 增加，运行时的损耗、Taro DOM Tree 的构建和更新、DOM data 初始化和更新。

而我们真正能做的，只有绿色部分，也就是：

-   **Taro DOM Tree 的构建和更新**
-   **DOM data 初始化和更新**

#### 精简的 DOM/BOM API

在 Taro DOM Tree 的构建和更新阶段，我们实现了一套仅实现了高效的、精简版 DOM/BOM API，而且仅仅实现了必要的。

Github 上有一个仓库 [jsdom](https://github.com/jsdom/jsdom "jsdom")，基本上是在 Node.js 上实现了一套 Web 标准的 DOM/BOM ，这个仓库的代码在压缩前大概有 2.1M，而 Taro Next 的核心的 DOM/BOM API 代码才 1000 行不到。

因此，我们最大限度的保证了 Taro DOM Tree 构建和更新阶段的性能。

![](https://img10.360buyimg.com/ling/jfs/t1/93372/24/9137/11878/5e0ae512E79c720f9/ab326eef51d168a5.jpg)

#### Taro3更新优化

无论是 React 还是 Vue ，最终都会调用 Taro DOM 方法，如：`appendChild`、`insertChild` 等。

这些方法在修改 Taro DOM Tree 的同时，还会调用 `enqueueUpdate` 方法，这个方法能获取到每一个 DOM 方法最终修改的节点路径和值，如：`{root.cn.[0].cn.[4].value: "1"}`，并通过 `setData` 方法更新到视图层。

![](https://img10.360buyimg.com/ling/jfs/t1/86893/14/9016/38824/5e0ac06aE21db32b1/c4bece64d139dd8e.jpg)

可以看到，这里**更新的粒度是 DOM 级别**，只有最终发生改变的 DOM 才会被更新过去，相对于之前 data 级别的更新会更加精准，性能更好。

> data更新是会有冗余的，并不是所有data更新会引起dom更新

#### 包体积优化

首先我们来看包 Size，下面的表格是 TodoMVC 的例子，在原生、Taro Old、Taro Next 等情况下的包大小对比，可以看到，引入 React/Vue 后，包大小在 Gzip 情况下大概增加了 30k 左右。

![](https://img12.360buyimg.com/ling/jfs/t1/100878/15/9005/22516/5e0ae322E690a60fb/9b3a09bf64a51871.jpg)

不过我们在前面一再强调：和之前模版通过编译生成的不同，Taro Next 的模版是固定的，然后基于组件的 template，动态 “递归” 渲染整棵 Taro DOM 树。也就是说，**Taro Next 的 WXML 大小是有上限的**。

随着项目的增加，页面越来越多，原生的项目 WXML 体积会不断增加，而 Taro Next 不会。也就是说，当页面的数量超过一个临界点时，Taro Next 的包体积可能会更小。因此，包 Size 的问题不足为虑。

![](https://img30.360buyimg.com/ling/jfs/t1/88749/23/9077/15347/5e0ae41cE71f77070/3f24bf6459abf2e9.jpg)
