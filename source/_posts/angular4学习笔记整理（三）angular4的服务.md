---
title: angular4学习笔记整理（三）angular4的服务
date: 2018-01-12 21:35:00
categories:
- angular
tags:
- service
---
## 依赖注入基本步骤
1. 生成service
```
export class ProductService {
constructor() { }
getProduct(): Product {
return new Product(1, 'iphone', 5899, 'hahah');
}
}
```
2. app.module.ts或者 某组件的配置里加上 提供器
```
@NgModule({
declarations: [
AppComponent,
Product1Component,
Product2Component
],
imports: [
BrowserModule
],
providers: [ProductService],
bootstrap: [AppComponent]
})
```
在module里配置的providers会对所有组件生效
但如果在组件的ts里面配置，则组件里配置的优先生效如：
```
@Component({
selector: 'app-product2',
templateUrl: './product2.component.html',
styleUrls: ['./product2.component.css'],
providers: [{provide: ProductService , useClass: AnotherProductService}]
})
```
并且可以根据
```
providers: [{provide: ProductService , useClass: AnotherProductService}]
```
来指定不同的服务，虽然不同class，但在组件生成的东西还是一样的

3. 在组件里面注入服务如
```
constructor(private productService: ProductService) { }
```

另外
服务之间也能相互注入靠@Injectable()
服务之间也可以注入，步骤一样，注册提供

还有这个
工厂和值申明提供器,高级玩法，我就学过没用过，提醒一下自己
```
providers: [
// ProductService,
{
provide: ProductService ,
useFactory: (logger: LoggerService, isDev) => {   //依赖注入服务工厂方法，实例化不同的productService
// let logger = new LoggerService();
if (isDev) {
return new ProductService(logger);
}else{
return new AnotherProductService(logger);
}
},
deps: [ LoggerService , "IS_DEV_ENV"]     //工厂方法也可以注入服务或者值，前一个是login服务 ，后一个是 变量注入（可以使对象）
},
{provide : "IS_DEV_ENV" , useValue : false },  //声明的值服务
LoggerService
],
```
