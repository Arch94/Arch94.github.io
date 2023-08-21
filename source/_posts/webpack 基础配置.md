---
title: webpack 基础配置
date: 2023-08-21 09:00:00
categories:
- webpack
tags:
- webpack
---

## 目录

-   [webpack 安装](#webpack-安装)
-   [webpack-cli 命令行打包](#webpack-cli-命令行打包)
-   [命令行修改打包输入输出目录](#命令行修改打包输入输出目录)
-   [命令行使用babel es6转es5](#命令行使用babel-es6转es5)
-   [Webpack 常见名词解释](#Webpack-常见名词解释)
-   [webpack配置文件](#webpack配置文件)
-   [mode 模式](#mode-模式)
-   [entry入口](#entry入口)
    -   [多文件入口](#多文件入口)
-   [output 输出](#output-输出)
-   [Loader](#Loader)
    -   [file-loader](#file-loader)
    -   [url-loader](#url-loader)
    -   [style-loader](#style-loader)
    -   [css-loader](#css-loader)
-   [Plugins](#Plugins)
    -   [HtmlWebpackPlugin](#HtmlWebpackPlugin)
    -   [CleanWebpackPlugin](#CleanWebpackPlugin)
-   [devServer](#devServer)
-   [模块热替换 HMR](#模块热替换-HMR)
-   [Babel es6转es5](#Babel-es6转es5)

### webpack 安装

`npm install webpack webpack-cli --save-dev`

创建目录并且进入

`mkdir zero-config && cd $_`初始化

`npm init -y` 快速生成package.json

安装 webpack 和 webpack-cli到开发依赖

`npm i webpack --save-dev `

`npm i webpack-cli --save-dev`

### webpack-cli 命令行打包

Webpack 默认的入口文件是src/index.js；

Webpack 的默认输出目录是dist/main.js。 &#x20;
\--mode development/production 配置打包测试/生产版本

```javascript
{
  "name": "zero-config",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "dev": "webpack --mode development",
    "build": "webpack --mode production"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^4.43.0",
    "webpack-cli": "^3.3.11"
  }
}
```

### 命令行修改打包输入输出目录

我们如果要修改 Webpack 的默认输出目录，需要用到 Webpack 命令的--output， 输入是 --entry 我们将上面的 npm scripts 做下修改：

```javascript
"scripts": {
  "dev": "webpack --mode development --entry ./src/app.js --output ./output/main.js",
}
```

### 命令行使用babel es6转es5

`npm i @babel/core babel-loader @babel/preset-env --save-dev`

创建.babelrc

```javascript
{
  "presets": ["@babel/preset-env"]
}
```

package.json修改

```javascript
"scripts": {
  "build": "webpack --mode production ./src/es/index.js --module-bind js=babel-loader"
}
```

### Webpack 常见名词解释

|         |                                                                 |
| ------- | --------------------------------------------------------------- |
| 参数      | 说明                                                              |
| entry   | 项目入口                                                            |
| output  | 打包文件输出                                                          |
| module  | 开发中每一个文件都可以看做 module，模块不局限于 js，也包含 css、图片等                      |
| chunk   | 代码块，一个 chunk 可以由多个模块组成                                          |
| loader  | 模块转化器，模块的处理器，对模块进行转换处理                                          |
| plugin  | 扩展插件，插件可以处理 chunk，也可以对最后的打包结果进行处理，可以完成 loader 完不成的任务            |
| bundle  | 最终打包完成的文件，一般就是和 chunk 一一对应的关系，bundle 就是对 chunk 进行便意压缩打包等处理后的产出  |
| devtool | source-map配置，默认在mode是development的时候是开着的，就能直接链到源代码，production就没有 |

### webpack配置文件

webpack默认配置文件为 webpack.config.js, 可以使用命令行 —config指定配置文件

`npx webpack --config webpack.config.entry.js --mode development`

### mode 模式

Webpack4.0 开始引入了mode配置，通过配置mode=development或者mode=production来制定是开发环境打包，还是生产环境打包，比如生产环境代码需要压缩，图片需要优化，Webpack 默认mode是生产环境，即mode=production

```javascript
module.exports = {
  mode: 'development'
};
```

还可以在命令行中设置mode：

```javascript
npx webpack --config webpack.config.entry.js --mode development
```

### entry入口

单文件的用法如下：

```javascript
module.exports = {
  entry: 'path/to/my/entry/file.js'
};
// 或者使用对象方式
module.exports = {
  entry: {
    main: 'path/to/my/entry/file.js'
  }
};
```

entry还可以传入包含文件路径的数组，当entry为数组的时候也会合并输出，例如下面的配置：

```javascript
module.exports = {
  mode: 'development',
  entry: ['./src/app.js', './src/home.js'],
  output: {
    filename: 'array.js'
  }
};
```

#### 多文件入口

多文件入口是使用对象语法来通过支持多个entry，多文件入口的对象语法相对于单文件入口，具有较高的灵活性，例如多页应用、页面模块分离优化。

```javascript
module.exports = {
  entry: {
    home: 'path/to/my/entry/home.js',
    search: 'path/to/my/entry/search.js',
    list: 'path/to/my/entry/list.js'
  }
};
```

上面的语法将entry分成了 3 个独立的入口文件，这样会打包出来三个对应的 bundle. &#x20;
对于一个 HTML 页面，我们推荐只有一个 entry ，通过统一的入口，解析出来的依赖关系更方便管理和维护

### output 输出

webpack 的output是指定了entry对应文件编译打包后的输出 bundle。output的常用属性是：

-   path：此选项制定了输出的 bundle 存放的路径，比如dist、output等,path路径可改&#x20;
    ```javascript
    path: path.resolve(__dirname , 'dist')
    ```
-   filename：这个是 bundle 的名称
-   publicPath：指定了一个在浏览器中被引用的 URL 地址

当不指定 output 的时候，默认输出到 dist/main.js

**一个 webpack 的配置，可以包含多个entry，但是只能有一个output。对于不同的entry可以通过output.filename占位符语法来区分**

```javascript
const path = require('path');
module.exports = {
  entry: {
    home: 'path/to/my/entry/home.js',
    search: 'path/to/my/entry/search.js',
    list: 'path/to/my/entry/list.js'
  },
  output: {
    filename: '[name].js',
    path: path.resolve(__dirname , 'dist')
  }
};
```

其中\[name]就是占位符，它对应的是entry的key（home、search、list），所以最终输出结果是：

```纯文本
path/to/my/entry/home.js → dist/home.js
path/to/my/entry/search.js → dist/search.js
path/to/my/entry/list.js → dist/list.js
```

Webpack 目前支持的占位符

|              |                           |
| ------------ | ------------------------- |
| 占位符          | 含义                        |
| \[hash]      | 模块标识符的 hash               |
| \[chunkhash] | chunk 内容的 hash            |
| \[name]      | 模块名称                      |
| \[id]        | 模块标识符                     |
| \[query]     | 模块的 query，例如，文件名 ? 后面的字符串 |

-   \[hash]：是整个项目的 hash 值，其根据每次编译内容计算得到，每次编译之后都会生成新的 hash，即修改任何文件都会导致所有文件的 hash 发生改变；在一个项目中虽然入口不同，但是 hash 是相同的；hash 无法实现前端静态资源在浏览器上长缓存，这时候应该使用 chunkhash；
-   \[chunkhash]：根据不同的入口文件（entry）进行依赖文件解析，构建对应的 chunk，生成相应的 hash；只要组成 entry 的模块文件没有变化，则对应的 hash 也是不变的，所以一般项目优化时，会将公共库代码拆分到一起，因为公共库代码变动较少的，使用 chunkhash 可以发挥最长缓存的作用；
-   \[contenthash]：使用 chunkhash 存在一个问题，当在一个 JS 文件中引入了 CSS 文件，编译后它们的 hash 是相同的。而且，只要 JS 文件内容发生改变，与其关联的 CSS 文件 hash 也会改变，针对这种情况，可以把 CSS 从 JS 中使用mini-css-extract-plugin 或 extract-text-webpack-plugin抽离出来并使用 contenthash。

### Loader

webpack 只能理解 JavaScript 和 JSON 文件，这是 webpack 开箱可用的自带能力。**loader 让 webpack 能够去处理其他类型的文件，并将它们转换为有效 **[**模块**](https://webpack.docschina.org/concepts/modules "模块")**，以供应用程序使用，以及被添加到依赖图中。**

在 webpack 的配置中，**loader** 有两个属性：

1.  `test` 属性，识别出哪些文件会被转换。
2.  `use` 属性，定义出在进行转换时，应该使用哪个 `loader`。同时在use 里配置loader时可以同时在旁边配置这个loader相应的`options`，对这个loader进行配置。

#### file-loader

```javascript
module.exports = {
  mode: 'development',
  entry: {
    main: './src/index.js'
  },
  module: {
    rules: [{
      test: /\.jpg$/,
      use: {
        loader: 'file-loader',
        //loader options
        options: {
          name: '[name].[ext]', // placeholder []占位符
          outputPath:'images/', 
        }
      } 
    }]
  },
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
}
```

#### url-loader

和file-loader 功能相似，不过对处理图片引用更加合适，可以根据图片大小配置不同的引用策略

```javascript
module.exports = {
  mode: 'development',
  entry: {
    main: './src/index.js'
  },
  module: {
    rules: [{
      test: /\.jpg$/,
      use: {
        loader: 'url-loader',
        //loader options
        options: {
          name: '[name]_[hash].[ext]', // placeholder []占位符
          outputPath:'images/', 
          limit:20480 //大于20kb的图片放在images，小于的放入bundle.js里转成base64
        }
      } 
    }]
  },
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
}
```

#### style-loader

css-loader负责整合css样式，style-loader负责将整合的样式挂载到html里的style标签里，配合使用

```javascript

module.exports = {
  mode: 'development',
  entry: {
    main: './src/index.js'
  },
  module: {
    rules: [{
      test:/\.css$/,
      use:['style-loader','css-loader']
    }]
  },
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
}
```

#### css-loader

重要配置项

| importLoaders | `importLoaders` 选项允许你配置在 `css-loader` 之前有多少 loader 应用于 `@import`ed 资源与 CSS 模块/ICSS 导入&#xA;（假如css-loader前有sass-loader和postcss-loader，如果不配置importLoaders，import导入的css模块就不会受这两个loader处理） |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| modules       | 启用/禁用 CSS 模块或者 ICSS 及其配置                                                                                                                                                              |

```javascript

module.exports = {
  mode: 'development',
  entry: {
    main: './src/index.js'
  },
  module: {
    rules: [{
      test: /\.jpg$/,
      use: {
        loader: 'url-loader',
        options: {
          name: '[name]_[hash].[ext]',
          outputPath: 'images/',
          limit: 2048
        }
      }
    }, {
      test: /\.css$/,
      use: ['style-loader', {
        loader:'css-loader',
        options:{
          importLoaders:2,
          modules:true
        }
      }, 'sass-loader', 'postcss-loader'],
    }]
  },
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
}
```

### Plugins

可以在webpack运行到某一时刻的时候，帮你做些什么

#### HtmlWebpackPlugin

会在打包结束后自动生成一个html文件，并把打包生成的js自动引入到这个html中

```javascript

module.exports = {
  ...
  plugins: [
    new HtmlWebpackPlugin({
      template: 'src/index.html' //配置生成html文件的模版
    }), 
  ],
  ...
}
```

#### CleanWebpackPlugin

在重新打包的时候把旧的打包文件删除

```javascript

module.exports = {
  ...
  plugins: [
    new HtmlWebpackPlugin({
      template: 'src/index.html' //配置生成html文件的模版
    }), 
    new CleanWebpackPlugin(['dist'])
  ],
  ...
}
```

### devServer

`webpack —watch` 也能实时监听打包文件变化，但没dev-server功能多

webpack-dev-server 不仅能实时监听，还能编译完重新刷新页面

注意：有没有发现平常开发时vue并没有生成dist目录，只有`npm run build`后才会生成dist，这是因为webpack-dev-server 将打包编译出来的文件放入内存中，从而提升它的打包速度，这是一个dev-server的隐藏特性

```javascript
npm i webpack-dev-server -d
```

```javascript
devServer:{
    //运行代码的目录
    contentBase:resolve(__dirname,"build"),
    //监视contentBase下的全部文件，一旦文件变化，就会reload
    watchContentBase:true,
    //监视中忽略某些文件
    watchOptions:{
        ignored:/node_modules/
    },
    //端口号
    port:3000,
    //域名
    host:'localhost',
    //启动gzip压缩
    compress:true,
    //自动打开浏览器
    open:true,
    //开启HMR功能
    hot:true,
    //不要启动服务的日志信息
    clientLogLevel:'none',
    //除了一些基本的启动信息以外，其他都不显示
    quiet:true,
    //如果出错了，不要全屏提示
    overlay:false,
    //服务器代理 -> 解决开发环境跨域问题
    proxy:{
        //一旦devServer5000接到/api/xxx的请求，就会把请求转发到另一个服务器3000
        '/api':{
            //转发后的目标地址
            // target: project_config.proxyAddr,
            target:'localhost:3000',
            // 发送请求时，请求路径重写 /api/xxx ->  /xxx （去掉a/pi）
            pathRewrite: {
                '^/api': ''
            }
        }
    }
}
```

### 模块热替换 HMR

作用：HMR 既避免了频繁手动刷新页面，也减少了页面刷新时的等待，可以极大地提高前端页面开发效率。

安装webpack插件HotModuleReplacementPlugin

这个插件是配合dev-server一起使用，如果只用dev-server ，在代码里改了样式什么，也会重新刷新页面。热更新插件能帮助webpack直接替换样式文件，不刷新页面状态。

devServer配置 hot：true ，插件里再加上hmr插件

```javascript
const devConfig = {
  mode: 'development',
  devtool: 'cheap-module-eval-source-map',
  devServer: {
    contentBase: './dist',
    open: true,
    port: 8080,
    hot: true //开启热更新
  },
  ...
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ],
}
```

![hmr实现原理](/img/image_GcvwJIz_Dd.png "hmr实现原理")

原理详细信息可以查看

<https://www.kancloud.cn/sllyli/webpack/1242354>

### Babel es6转es5

```bash
npm install --save-dev babel-loader @babel/core
npm install @babel/preset-env --save-dev  //这个才是es6转es5用的东西，里面有es6转es5的翻译规则
npm install --save @babel/polyfill


```

```javascript
{
  module: {
    rules: [
      {
        test: /\.m?js$/,
        exclude: /node_modules/,//这个一定要有，因为node_modules里都是依赖包
        use: {
          loader: "babel-loader",
          options: {
            presets: ['@babel/preset-env']
          }
        }
      }
    ]
  }
}
```

babel的配置项可能会很多，可以新建一个`.babelrc`文件专门存放配置

```bash
{
  presets: [
    [
      "@babel/preset-env", {
        targets: {
          chrome: "67",
        },
        useBuiltIns: 'usage'
      }
    ],
    "@babel/preset-react"
  ],
  plugins: ["@babel/plugin-syntax-dynamic-import"]
}
```
