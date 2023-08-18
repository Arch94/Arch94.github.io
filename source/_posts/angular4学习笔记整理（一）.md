---
title: angular4学习笔记整理（一）
date: 2018-01-12 21:35:00
categories:
- angular
---
好久没写点东西了，毕业刚刚回到上海，就马上出差去了杭州去做app h5的webview开发，用的是ionic3，ionic现在学习还没学完，现在就把自己学到的angular的笔记整理一下。
## angular 的 nodejs安装
```
npm install -g angular
npm -g install @angular/cli    angular命令行工具安装
```
用typescript来写node需要引入一个包
```
npm i @types/node --save
```
## angular cli常用的command
ng new 项目名称          新建angular项目
ng g component xxx  生成组件
ng g service xxx  生成服务     
## jquery的使用
其实angular4里可以使用jquery了，不用像angular1里面使用类似jq的元素选择器，不过其中需要一点配置。
1. npm安装jq 
```
npm install jquery --save
```
2. 在angular-cli.json  里的app 配置文件里面 的styles、scripts里面添加js或者css的相对路径,就放在app的script里面，里面还有一个框的是bootstrap
![Screenshot 2018-01-12_19-03-46.png](http://upload-images.jianshu.io/upload_images/6114493-648480aa2acad151.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 由于typescript 不认识js的东西，还需要一个 类型描述文件
```
npm install @types/jquery --save
```
## 指令的简单运用
像下面这样一段代码基本即看懂，*ngFor循环数组 ，js的class样式控制用这种[class.xxxx]="xx" 这种方式
```
html
<p >
<span *ngFor="let star of stars" class="glyphicon glyphicon-star"
[class.glyphicon-star-empty]="star"></span>
{{rating}}
</p>
ts
@Component({
selector: 'app-stars',
templateUrl: './stars.component.html',
styleUrls: ['./stars.component.css']
})

export class StarsComponent{
private rating: number = 0;
private stars: boolean[];
constructor() { }
}
```
## 父组件向子组件传递数据 
子组件代码就是上面一段代码，它需要父组件输入一个rating属性，父组件调用子组件，只需在html里面调用子组件的html标签，输入属性用[]扩起来
```
<app-stars [rating]="product.rating"></app-stars>
```
输入属性：这种属性绑定  是输入数据的绑定方式

然后子组件里面的代码也需要改一下，将rating设置为输入属性
```
export class StarsComponent implements OnInit {
@Input()        通过这个input输入标注     声明rating会被父组件输入的属性覆盖
private rating: number = 0;//默认
}
```

