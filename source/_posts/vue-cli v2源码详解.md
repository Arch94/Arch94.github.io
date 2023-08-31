---
title: vue-cli v2源码详解
date: 2023-08-31 09:33:00
categories:
- vue
tags:
- vue
---

### 总结

#### vue init \<template-name> \<project-name> 执行过程分析

1.  获取参数
    ```javascript
    /**
     * Usage.
     * 从命令中获取参数
     * program.args[0]  模板类型
     * program.args[1]  自定义项目名称
     * program.clone    clone
     * program.offline  离线
     */
    program
      .usage('<template-name> [project-name]')
      .option('-c, --clone', 'use git clone')
      .option('--offline', 'use cached template')
    ```
2.  获取模板路径
    ```javascript
    // rawName存在或者为“.”的时候，视为在当前目录下构建
    const inPlace = !rawName || rawName === '.'
    // path.relative（）:根据当前工作目录返回相对路径
    const name = inPlace ? path.relative('../', process.cwd()) : rawName
    // 合并路径
    const to = path.resolve(rawName || '.')
    // 检查参数是否clone
    const clone = program.clone || false
    // path.join（）:使用平台特定分隔符,将所有给定的路径连接在一起,然后对结果路径进行规范化
    // 如 ： /Users/admin/.vue-templates/webpack
    const tmp = path.join(home, '.vue-templates', template.replace(/[\/:]/g, '-'))
    ```
3.  run 函数：区分本地和离线，下载模板
    ```javascript
    // 本地路径存在
    if (isLocalPath(template)) {
      // 获取绝对路径
      const templatePath = getTemplatePath(template)
      // 本地下载...
    } else {
         // 不包含“/”，去官网下载
        if (!hasSlash) {
          const officialTemplate = 'vuejs-templates/' + template
        // 包含“/”，去自己的仓库下载
        } else {
        }
    }
    ```

#### generate方法模板渲染的流程：

1.  **获取模板配置，**
2.  **初始化 Metalsmith，添加变量至 Metalsmith**
3.  handlebars 模板注册 helper
4.  **执行 before 函数(如果有)**
5.  **询问问题，过滤文件，渲染模板文件**
6.  **执行 after 函数(如果有)**
7.  构建项目
8.  **构建完成后，有 complete 函数则执行**，没有则打印配置对象中的 completeMessage 信息，有错误就执行回调函数 done(err)

### 参考

<https://xie.infoq.cn/article/799d4039602c89a2262ad55d9>
