---
title: angular4学习笔记整理（四）组件间通讯、管道
date: 2018-01-12 21:35:00
categories:
- angular
tags:
- pipe 
- emit
---
####组件间通讯
1.组件间通讯 。  
父组件向子组件输入属性用
```
<app-order [stockCode]="stock" [amount]="100"></app-order>
```
子组件声明接收父组件的属性@input()注解
```
@Input()
amount: number;
```

2.组件输出属性

1.在发射的组件内部定义发射的EventEmitter对象
```
@Output()
lastPrice: EventEmitter<PriceQuote> = new EventEmitter();
```
2.在发射组件里 将要发射的变量发射出去 ，注意类型必须和定义里的泛型一致
```
let pq = new PriceQuote(this.stockCode, Math.random() * 100);
this.lastPrice.emit(pq);
```
3.在发射组件标签声明的地方加上监听该emitEvent对象传过来的事件
```
<app-price-quote (lastPrice)="priceQuoteHandler($event)"></app-price-quote>
```

然后父组件里就可以写，事件就是发射过来的值
```
priceQuoteHandler(priceQuote) {
this.priceQuote = priceQuote;
}
```
注意想监听事件的名字即不是lastPrice 只要在output里改即可
```
@Output('priceChange')
```
但是这样有感觉很麻烦，能不能用**双向绑定**

还有注意 如果一个属性想用双向绑定 那么如果输入属性为rating ，并且想在标签上```[(rating)]```，获取输出值那么在组件内的输出属性 名称必须为```ratingChange```  ，加个Change
```
@Output()
private ratingChange: EventEmitter<number> = new EventEmitter();
```

####管道
普通应用，这个使用可以去查官网
```
<p>{{birthday | date : 'yyyy-MM-dd HH:mm:ss'}}</p>
<p>{{pi | number: '2.2-4'}}</p>
```
自定义管道需命令行生成 ```ng g pipe name```
管道和组件一样需申明在NgModule里
```
declarations: [
FilterPipe
],
```
自定义管道
```
import { Pipe, PipeTransform } from '@angular/core';
@Pipe({
name: 'filter'
})
export class FilterPipe implements PipeTransform {
transform(list: any[], filterField: string, keyword: string): any {
if (!filterField || !keyword) {
return list;
}
return list.filter( item => {
let fieldVaule = item[filterField];
return fieldVaule.indexOf(keyword) >= 0;
});
}
}
```
####如何在父组件中调用子组件的方法
子组件就一个gretting的方法
```
父组件html代码
<app-children #child1></app-children>
<app-children #child2></app-children>
<button (click)="child2.gretting('jsex')">hahah1</button>
```
父组件ts代码
```
import { Component, OnInit, ViewChild} from '@angular/core';
import {ChildrenComponent} from './children/children.component';

@Component({
selector: 'app-root',
templateUrl: './app.component.html',
styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit,{
@ViewChild('child1')
child1: ChildrenComponent;
greeting: string = 'heelo';
ngOnInit(): void {
this.child1.gretting(this.greeting);
}
}
```

1.在ts中调用子组件方法
html里
```
<app-children #child1></app-children>
```
ts里
```
@ViewChild('child1')
child1: ChildrenComponent;
```
父组件ts任意地方就可以
```
this.child1.gretting(this.greeting);
```

2.在html里调用子组件api
```
<app-children #child2></app-children>
<button (click)="child2.gretting('jsex')">hahah</button>
```

