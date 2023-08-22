---
title: webpack 高级概念
date: 2023-08-22 09:00:00
categories:
- webpack
tags:
- webpack
---

## 目录

-   [Tree shaking ](#Tree-shaking-)
-   [测试生产环境打包模式区分](#测试生产环境打包模式区分)
-   [code spliting](#code-spliting)
-   [懒加载、预加载](#懒加载预加载)
-   [Caching缓存](#Caching缓存)
-   [环境变量](#环境变量)

### Tree shaking&#x20;

作用：消除无用代码

在 webpack 项目中，有一个入口文件，相当于一棵树的主干，入口文件有很多依赖的模块，相当于树枝。实际情况中，虽然依赖了某个模块，但其实只使用其中的某些功能。通过 Tree-Shaking，将没有使用的模块摇掉，这样来达到删除无用代码的目的。

配置方法：

mode：production 打包模式时默认使用，mode：development时默认不配置

### 测试生产环境打包模式区分

这个很常见，主要通过打包命令来指定不同的运行环境或配置文件，举一个常见的

```bash
package.json
"dev": "webpack-dev-server --config ./build/webpack.common.js",
"build": "webpack --env.production --config ./build/webpack.common.js"

```

```javascript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CleanWebpackPlugin = require('clean-webpack-plugin');
const webpack = require('webpack');
const merge = require('webpack-merge');
const devConfig = require('./webpack.dev.js');
const prodConfig = require('./webpack.prod.js');
const commonConfig = {
  entry: {
    main: './src/index.js',
  },
  ....
  output: {
    path: path.resolve(__dirname, '../dist')
  }
}
//通过模式判断合并不同的配置，完成不同的打包效果
module.exports = (env) => {
  if(env && env.production) {
    return merge(commonConfig, prodConfig);
  }else {
    return merge(commonConfig, devConfig);
  }
}
```

### code spliting

代码分割，相比于入口文件配置的代码分割，splitChunks更具策略

```javascript
  const commonConfig = {
    ...
    optimization: {
      runtimeChunk: {
        name: 'runtime'
      },
      usedExports: true,
      splitChunks: {
        // 自动提取所有公共模块到单独 bundle
        chunks: 'all',
        cacheGroups: {
          // vendors这个键名即是打完包的前缀名
          //属性名可以自己取，只要不跟缓存组下其他定义过的属性同名就行，否则后面的拆分规则会把前面的配置覆盖掉
          vendors: {
            //test即为测试正则，当引用的模块名满足了这个测试正则后，才会按照当前对象的规则打包
            test: /[\\/]node_modules[\\/]/,
            priority: -10,
            name: 'vendors',
          },
          default: {
            minChunks:2, //模块最少引用数
            priority: -20,
          }
        }
      }
    },
    ...
  }
```

\*\*- chunks  \*\*

initial、async、all分别表示静态引入、动态引入和两种情况下来进行代码分割。

**- priority** &#x20;

priority属性的值为数字，可以为负数。作用是当缓存组中设置有多个拆分规则，而某个模块同时符合好几个规则的时候，则需要通过优先级属性priority来决定使用哪个拆分规则。优先级高者执行。我这里给业务代码组设置的优先级为-20，给第三方库组设置的优先级为-10，这样当一个第三方库被引用超过2次的时候，就不会打包到业务模块里了。

**- minChunks** &#x20;
就是在entry中配置的入口，引用了该模块的最少个数，只有满足了最少有minChunks个chunk引用了当前模块，才会进行单独打包

### 懒加载、预加载

事实上懒加载不是webpack的功能，它是es6的模块功能

```javascript
//懒加载
import(/*webpackChunkName:'test' */"./test")
  .then(({test}) => {
      console.log('test加载成功')
      test()
  })
  .catch(error => {
      console.log('test加载失败 error:', error)
  })
//预加载
import(/*webpackChunkName:'test' ,webpackPrefetch:true*/"./test")
  .then(({test}) => {
      console.log('test加载成功')
      test()
  })

```

### Caching缓存

***为什么需要hash？***

浏览器会缓存我们的文件。

优点是浏览器读取缓存的文件，能带来更佳的用户体验（不需要额外流量，速度更快）；

缺点是有时候我们修改了文件内容，但是浏览器依然读取缓存的文件（也就是旧文件），导致用户看到的文件不是最新的。

解决方法：文件尾加哈希，防止文件更新后缓存不刷新。文件内容不变化，哈希不会变。

```javascript
output: {
    filename: '[name].[contenthash].js',
    chunkFilename: '[name].[contenthash].js'
  }
```

### 环境变量

在打包命令里可以配置

```bash
package.json
"build": "webpack --env.production --config ./build/webpack.common.js"

```

```bash
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CleanWebpackPlugin = require('clean-webpack-plugin');
const webpack = require('webpack');
const merge = require('webpack-merge');
const devConfig = require('./webpack.dev.js');
const prodConfig = require('./webpack.prod.js');
const commonConfig = {
...
}
//环境变量
module.exports = (env) => {
  if (env && env.production) {
    return merge(commonConfig, prodConfig);
  } else {
    return merge(commonConfig, devConfig);
  }
}
```

也可以直接通过打包模式区分

```bash
if (process.env.NODE_ENV === 'production') {
  console.log('你正在线上环境');
} else {
  console.log('你正在使用开发环境');
}
```
