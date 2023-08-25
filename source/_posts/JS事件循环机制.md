---
title: JS事件循环机制
date: 2023-08-25 09:33:00
categories:
- js
tags:
- Event Queue
---

## 目录

-   [同步任务、异步任务](#同步任务异步任务)
-   [任务类型：宏任务、微任务](#任务类型宏任务微任务)

#### 同步任务、异步任务

JS 里的一种任务分类方式分为: 同步任务和异步任务

同步和异步任务分别进入不同的执行环境，**同步的进入主线程**，即主执行栈，异步的进入任务队列 Event Queue 。

**当同步任务执行完毕，会去任务队列执行相应的异步任务，推入主线程执行**。上述过程的不断重复就是我们说的 Event Loop (事件循环)。

![](/img/image_lkYCeF8KVJ.png)

#### 任务类型：宏任务、微任务

口诀：有微则微，无微则宏

每一次执行完宏任务后，都会清空微任务，再执行下一个宏任务

宏任务：script(整体代码)、setTimeout、setInterval、UI 渲染、 I/O、postMessage、 MessageChannel、setImmediate(Node.js 环境)

微任务：Promise、 MutaionObserver、process.nextTick(Node.js环境）

![](/img/image_gF4Ud4p1GT.png)
