---
title: 有用的css知识收集
date: 2017-12-07 14:42:12
categories:
- css
---

**currentColor**
表示“当前的标签所继承的文字颜色”。

**实战1：背景色镂空技术**去年介绍过“[CSS背景色镂空技术](http://www.zhangxinxu.com/wordpress/?p=3545)”，可以方便控制图标的颜色，很赞的想法。此文章对应demo可以[轻戳这里](http://www.zhangxinxu.com/study/201307/background-color-insert-background-image.html)访问。
这种设计的目的就是鼠标hover时候，图标可以跟着文字一起变色。如果不考虑兼容性问题，我们可以稍稍改造下，使其实现更加简单：
```sh
.icon { 
display: inline-block; width: 16px;
height: 20px; 
background-image: url(../201307/sprite_icons.png); 
background-color: currentColor; /* 该颜色控制图标的颜色 */
}
```
于是，我们想要鼠标hover文字链接，其图标颜色要跟着一起变化，只要改变文字颜色就可以了.
```sh
.link:hover { color: #333; }/* 虽然改变的是文字颜色，但是图标颜色也一起变化了 */
```
HTML结构如下：
``<a href="##" class="link"><i class="icon icon1"></i>返回</a>``
于是鼠标hover就是``#333``
因为``currentColor``继承文本颜色这个属性,``color``的改变会随之改变``background-color``

## meta
---
` <meta http-equiv="x-ua-comatible" content="ie=edge">`
1.  它代表ie文档的兼容性，告诉文档在**ie**下的兼容模式，
2.  它是为了兼容一些在ie8下显示不正常，但在老版本浏览器下显示正常的模式
3.  通过content  可以告诉 ie浏览器 你可以模拟 ie7的形式或者ie8或者ie9、ie11的形式显示网页
4.  比如`content="IE=EmulateIE8"`以ie8的模式渲染页面
5.  ie=edge是为了强制ie浏览器以最新的模式渲染页面，能多新，有多新，**（但如果浏览器最高ie8，那也只能用ie8的模式渲染）**。

## 移动端meta必备
`<meta name="view-port" content="width=device-width,initial-scale=1">`

## 对于ie低版本兼容
- css里 \0  大家都懂
- html里
```sh
<!--[if lte IE8]>
<p>如果浏览器小于等于ie8，那么我提示你该升级了</p>
<![endif]-->
```
- 格式不多说  gt 大于、lt小于、gte大于等于、lte小于等于


## px em rem
1. px像素
2. em 相对父元素 ，如果没设会一直往上找，很强大但会导致混乱
3. rem 相对html，但rem ie678不支持

```sh
html {
font-size: 62.5%;
color: #222;
}
```
如此一个rem会是10px 

## 取消选中
css3属性  顺序不能乱，不然谷歌没有，火狐有
```sh
::-moz-selection{
background-color: #b3d4fc;
text-shadow: none;
}
::selection{
background-color: #b3d4fc;
text-shadow: none;
}
```
## 隐藏文字
```sh
.text-hide{
font:0/0 a;
color: transparent;
text-shadow: none;
background-color: transparent;
border: 0;
}
```
注意其中的text-hide 是为了隐藏文字，方便seo识别图片

## 清除浮动
```sh
.clearfix:after,.clearfix:before{
content:' ';
display: table;
}
.clearfix:after{
clear:both;
}
```
这种方法可以防止margin的叠加



## before 利用
```sh
.notice a:first-child:before{
content: '最新公告：\00a0\00a0';
color: #aaa;
}
```
其中`\00a0\00a0`是不换行的空白字符,因为content没法用`&nbsp`添加空格

## 文字不换行,多出省略
```sh
.notice a:first-child{
text-overflow: ellipsis;
overflow: hidden;
white-space: nowrap;
}
```

## 设备信息
用户代理字符串在控制台输入`navigator.userAgent`获得相关设备信息

## 文字超出隐藏
``` sh
{
overflow: hidden;
white-space: nowrap;
text-overflow: ellipsis;
}
```
## 图标随文字变色设计
**背景色镂空技术** 利用 
**currentColor**表示“当前的标签所继承的文字颜色”。
这种设计的目的就是鼠标hover时候，图标可以跟着文字一起变色。
```sh
.icon {
display: inline-block;
width: 16px; height: 20px;
background-image: url(../201307/sprite_icons.png);
background-color: currentColor; /* 该颜色控制图标的颜色 */
}
```
于是，我们想要鼠标hover文字链接，其图标颜色要跟着一起变化，只要改变文字颜色就可以了：
```sh
.link:hover { 
color: #333; 
}
```

因为``currentColor``继承文本颜色这个属性,``color``的改变会随之改变``background-color``

## margin-top溢出问题：
给元素盒子一个垂直外边距margin-top，父元素盒子也会往下走margin-top的值。
有以下几点解决方案
1.修改父元素高度，增加padding-top                     好用，但间距变成margin+padding
2.为父元素添加overflow：hidden                            特别好用，相当于子元素向下移，但范围和子元素大小要注意
3.为父元素或者子元素申明浮动（float：left）            可以用，但会改变父子的定位，会浮起来
4.为父元素添加border                                            好用，但会改变边框

## 毛玻璃特效
css代码
```sh
.blur {    
-webkit-filter: blur(20px); /* Chrome, Opera */
-moz-filter: blur(20px);
-ms-filter: blur(20px);    
filter: blur(20px);    
}
/**--图片上的div--**/
.top-bg{
position: absolute;
top: 0;
left: 0;
width: 100%;
height: 100%;
z-index: -1;
overflow: hidden;
}
.top-bg img{
transform: scale(4.0);
}
```
html代码
```sh
<div>  <!--父层-->
<div class="top-bg blur"> <!--背景层-->
![](./img/37403260_p0.png)  <!-- 图片层-->
</div>
</div>
```
