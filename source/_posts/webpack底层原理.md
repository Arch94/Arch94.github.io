---
title: webpack底层原理
date: 2023-08-23 09:42:00
categories:
- webpack
tags:
- webpack
---

## 目录

-   [如何编写一个Loader](#如何编写一个Loader)
    -   [同步loader实现](#同步loader实现)
    -   [异步loader实现](#异步loader实现)
    -   [config配置自定义loader](#config配置自定义loader)
-   [如何编写一个Plugin](#如何编写一个Plugin)
-   [webpack 执行流程](#webpack-执行流程)

### 如何编写一个Loader

#### 同步loader实现

```javascript
const loaderUtils = require("loader-utils");
module.exports = function(source){ //source是原文本
  const options = loaderUtils.getOptions(this); //拿到option参数，其实就是this.query
  console.log(source)
  console.log(options)
  return source.replace('1234','5678') //最简单的loader，文本替换就完了
}
```

如果有错误信息，要抛出去怎么解决呢,调用 this.callback即可

```javascript
 this.callback('错误信息',source.replace('1234','5678'))
```

#### 异步loader实现

```javascript
const loaderUtils = require('loader-utils');
module.exports = function(source) {
  const options = loaderUtils.getOptions(this);
  const callback = this.async();

  setTimeout(() => {
    const result = source.replace('dell', options.name);
    callback(null, result);
  }, 1000);
}
```

#### config配置自定义loader

```javascript
const path = require('path');

module.exports = {
  mode: 'development',
  entry: {
    main: './src/index.js'
  },
  resolveLoader: { //一定要配置自定义loader目录，不然自定义loader就得path引入
    modules: ['node_modules', './loaders']
  },
  module: {
    rules: [{
      test: /\.js/,
      use: [
        {
          loader: 'customLoader', //自定义loader引入
          options: {
            name: '2234'
          }
        }
      ]
    }]
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].js'
  }
}
```

### 如何编写一个Plugin

plugin 的实现可以是一个类，使用时传入相关配置来创建一个实例，然后放到配置的 `plugins` 字段中，而 plugin 实例中最重要的方法是 `apply`，该方法在 webpack compiler 安装插件时会被调用一次，`apply` 接收 webpack compiler 对象实例的引用，你可以在 compiler 对象实例上注册各种事件钩子函数，来影响 webpack 的所有构建流程，以便完成更多其他的构建任务。

> compiler 对象代表了完整的 webpack 环境配置。这个对象在启动 webpack 时被一次性建立，并配置好所有可操作的设置，包括 options，loader 和 plugin。当在 webpack 环境中应用一个插件时，插件将收到此 compiler 对象的引用。可以使用它来访问 webpack 的主环境。

> compilation 对象代表了一次资源版本构建。当运行 webpack 开发环境中间件时，每当检测到一个文件变化，就会创建一个新的 compilation，从而生成一组新的编译资源。一个 compilation 对象表现了当前的模块资源、编译生成资源、变化的文件、以及被跟踪依赖的状态信息。compilation 对象也提供了很多关键时机的回调，以供插件做自定义处理时选择使用

```javascript
class CopyrightWebpackPlugin {
  constructor(options){
    //options插件传参
    console.log(options)
  }
  //核心方法
  apply(compiler) { // compiler webpack实例
    //hooks 在emit 阶段触发
    compiler.hooks.emit.tapAsync('CopyrightWebpackPlugin', (compilation, cb) => {
      //打包生成新文件
      compilation.assets['copyright223.txt']= {
        source: function() {
          return 'copyright by sunqixiong'
        },
        size: function() {
          return 21;
        }
      };
      //异步方法要执行cb进行回调
      cb();
    })
  }

}

module.exports = CopyrightWebpackPlugin;
```

```javascript
const path = require('path');
const CopyRightWebpackPlugin = require('./plugins/copyright-webpack-plugin'); //本地引入

module.exports = {
  mode: 'development',
  entry: {
    main: './src/index.js'
  },
  plugins: [
    new CopyRightWebpackPlugin({name:1}) //传参
  ],
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].js'
  }
}
```

### webpack 执行流程

![webpack 执行过程](/img/image_t4A6GJsauX.png "webpack 执行过程")

`ebpack`包含两个很重要的基础概念,分别是`compiler`和`compilation`.

`compiler`是`webpack`的支柱引擎,它相当于系统中枢,控制着程序的执行.

代码头部首先引入`webpack`和配置文件参数`options`,通过执行`webpack(options)`即可生成`compiler`对象,再执行对象的`run`方法就能开始启动代码编译.

```javascript
const webpack = require("webpack");
const options = require("../webpack.config.js");

const compiler = webpack(options);
compiler.run(); // 启动代码编译
```

上图中,当`compiler`执行`make`阶段时,标志着代码的编译工作正式开始,这时候会创建`compilation`对象完成相关任务.

`compilation`会依次执行第二行标记的`3`个钩子,等到代码的编译工作结束后,主线程又回到了`compiler`,继续往下执行`emit`钩子.

简而言之,`compiler`**执行到**\*\*`make`****和****`emit`\*\***之间时**,`compilation`对象便出场了,它会依次执行它定义的一系列钩子函数,像代码的编译、依赖分析、优化、封装正是在这个阶段完成的.

`compilation`实例主要负责代码的编译和构建.每进行一次代码的编译(例如日常开发时按`ctrl + s`保存修改后的代码),都会重新生成一个`compilation`实例负责本次的构建任务.

整体执行流程已经梳理了一遍,接下来深入到上图中标记的每一个钩子函数,理解其对应的时间节点.

-   `entryOption:``webpack`开始读取配置文件的`Entries`,递归遍历所有的入口文件.
-   `run:` 程序即将进入构建环节
-   `compile:` 程序即将创建`compilation`实例对象
-   `make:``compilation`实例启动对代码的编译和构建
-   `emit:` 所有打包生成的文件内容已经在内存中按照相应的数据结构处理完毕,下一步会将文件内容输出到文件系统,`emit`钩子会在生成文件之前执行(通常想操作打包后的文件可以在`emit`阶段编写`plugin`实现).
-   `done:` 编译后的文件已经输出到目标目录,整体代码的构建工作结束时触发

`compilation`下的钩子含义如下.

-   `buildModule:` 在模块构建开始之前触发,这个钩子下可以用来修改模块的参数
-   `seal:` 构建工作完成了,`compilation`对象停止接收新的模块时触发
-   `optimize:` 优化阶段开始时触发

`compiler`进入`make`阶段后,`compilation`实例被创建出来,它会先触发`buildModule`阶段定义的钩子,此时`compilation`实例依次进入每一个入口文件(`entry`),加载相应的`loader`对代码编译.

代码编译完成后,再将编译好的文件内容调用 `acorn` 解析生成`AST`语法树,按照此方法继续递归、重复执行该过程.

所有模块和和依赖分析完成后,`compilation`进入`seal` 阶段,对每个`chunk`进行整理,接下来进入`optimize`阶段,开启代码的优化和封装.

<https://zhuanlan.zhihu.com/p/400015634>
