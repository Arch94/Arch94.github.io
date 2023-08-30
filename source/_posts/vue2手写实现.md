---
title: vue2手写实现
date: 2023-08-30 10:42:00
categories:
- vue
tags:
- vue
---

![](</img/iShot2021-01-22 16.11.32_DbQLo_urMf.png>)

#### 问什么是vue？

vue是一套开源的用于构建用户界面的渐进式MVVM框架，通过compile来解析模版指令，通过Observer劫持监听数据变化，通过dep收集依赖，通过watcher更新视图。

mvue.js

```javascript
class mvue {
  constructor(options){
    this.$el = options.el;
    this.$data = options.data;
    this.$options = options;
    if(this.$el){
      new Observer(this.$data);
      new Compile(this.$el,this);
      this.proxyData(this.$data);
    }
  }
  proxyData(data){
    for(const key in data){
      Object.defineProperty(this,key,{
        get(){
          return data[key];
        },
        set(newVal){
          data[key] = newVal;
        }
      })
    }
    
  }
}

class Compile{
  constructor(el,vm){
    this.el = this.isElementNode(el) ? el : document.querySelector(el)
    this.vm = vm;
    const fragment = this.node2Fragment(this.el);
    this.compile(fragment);
    this.el.appendChild(fragment);
  }
  isElementNode(node){
    return node.nodeType === 1;
  }
  node2Fragment(el){
    let fragment = document.createDocumentFragment();
    let firstChild;
    while(firstChild = el.firstChild){
      fragment.appendChild(firstChild);
    }
    return fragment;
  }
  compile(fragment){
    let childNodes = fragment.childNodes;
    //转数组
    [...childNodes].forEach(childNode => {
       if(this.isElementNode(childNode)){
         this.compileElement(childNode);
       } else {
         this.compileText(childNode);
       }
       //递归子元素的子元素
       if(childNode.childNodes && childNode.childNodes.length){
        this.compile(childNode);
       }
    })
  }
  compileElement(node){
    const attributes = node.attributes;
    [...attributes].forEach(attr=>{
      const {name,value} = attr;
      if(this.isDirective(name)){//是一个指令 v-text v-html v-on:click 
        const [,directive] =  name.split('-'); 
        const [dirName,eventName] = directive.split(':') //v-on:click  v-bind:href
        //更新数据，数据驱动视图
        compileUtil[dirName](node,value,this.vm,eventName)
        //删除标签上原有的指令
        node.removeAttribute('v-'+directive);
      } else if(this.isEventName(name)){//@click="xxxs"
        const eventName = name.split('@')[1];
        //更新数据，数据驱动视图
        compileUtil['on'](node,value,this.vm,eventName);
      }
    })
  }

  isEventName(attrName){
    return attrName.indexOf('@') == 0;
  }

  compileText(node){
    const content = node.textContent;
    if(/\{\{(.+?)\}\}/.test(content)){
      compileUtil['text'](node,content,this.vm)
    }
  }

  isDirective(name){
    return name.startsWith('v-')?true:false;
  }
}

const compileUtil = {
  getVal(expr,vm){
    return expr.split('.').reduce((data,currentVal)=>{
      return data[currentVal]
    },vm.$data)
  },
  setVal(expr,vm,inputVal){
    expr.split('.').reduce((data,currentVal)=>{
      data[currentVal] = inputVal
    },vm.$data)
  },
  getContentVal(expr,vm){
    return expr.replace(/\{\{(.+?)\}\}/gi,(...args) => {
      return this.getVal(args[1],vm);
    })
  },
  text(node,expr,vm){
    let value;
    if(expr.indexOf('{{')!==-1){
      value = expr.replace(/\{\{(.+?)\}\}/gi,(...args) => {
        new Watcher(vm,args[1],()=>{
          this.updater.textUpdater(node,this.getContentVal(expr,vm));
        })
        return this.getVal(args[1],vm);
      })
    } else {
      value = this.getVal(expr,vm);
      new Watcher(vm,expr,newVal=>{
        this.updater.textUpdater(node,newVal);
      })
    }
    this.updater.textUpdater(node,value);
  },
  html(node,expr,vm){
    //1. getVal 触发 observer数据监听器的 get方法，没有 Dep.target
    const value = this.getVal(expr,vm);
    // 2. 初始化视图
    this.updater.htmlUpdater(node,value);
    //3.新建 观察者，初始化内部调用compileUtil.getVal获取旧值，并将Dep.target设置为此watcher
    //4.再次触发 observer数据监听器的 get方法，将该观察者加入到 这个expr值的订阅器Dep中
    new Watcher(vm,expr,newVal=>{
      this.updater.htmlUpdater(node,newVal);
    })
    //5.在 控制台 vm.$data.msg2 = 'xdddd' 触发数据监听器 dep.notify => dep.update => 更新视图
  },
  model(node,expr,vm){
    const value = this.getVal(expr,vm);
    this.updater.modelUpdater(node,value);
    //数据=》视图
    new Watcher(vm,expr,newVal=>{
      this.updater.modelUpdater(node,newVal);
    })
    //视图=》数据=》视图
    node.addEventListener('input',(e)=>{
      this.setVal(expr,vm,e.target.value)
    })
  },
  on(node,expr,vm,eventName){
    console.log(node,expr,vm,eventName)
    let fn = vm.$options.methods && vm.$options.methods[expr];
    node.addEventListener(eventName,fn.bind(vm),false);
  },
  bind(node,expr,vm,attrName){
    //do self
    console.log(node,expr,vm,attrName)
    let value = this.getVal(expr,vm);
    if(attrName === 'class'){//实际css设置更加复杂，expr 不仅会是一个对象，也会是[] {} 的对象
      Object.assign(node.style,value)
    } else {
      node.setAttribute(attrName,value)
    }
  },
  updater:{
    textUpdater(node,value){
      node.textContent = value;
    },
    htmlUpdater(node,value){
      node.innerHTML = value;
    },
    modelUpdater(node,value){
      node.value = value;
    }
  }
}
```

observer.js

```javascript
class Watcher {
  constructor(vm,expr,cb){
    this.vm = vm;
    this.expr = expr;
    this.cb = cb;
    //旧值保存
    this.oldVal = this.getOldVal();
  }
  getOldVal(){
    //指定target
    Dep.target = this;
    const oldVal = compileUtil.getVal(this.expr,this.vm);
    //清除target
    Dep.target = null;
    return oldVal;
  }
  update(){
    const newVal = compileUtil.getVal(this.expr,this.vm);
    if(newVal !== this.oldVal){
      this.cb(newVal);
    }
  }
}

class Dep {
  constructor(){
    this.subs = [];
  }
  //添加观察者
  addSub(watcher){
    this.subs.push(watcher);
  }
  //通知观察者去更新
  notify(){
    this.subs.forEach(w=>w.update())
  }
}

class Observer {
  constructor(data){
    this.observe(data)
  }
  /**
   * person:{
      name:'111',
      age:'26'
    },
    msg:'model dire',
   */
  observe(data){
    if(data && typeof data === 'object'){
      Object.keys(data).forEach(key=>{
        this.defineReactive(data,key,data[key])
      })
    }
  }
  
  defineReactive(data,key,value){
    //递归遍历
    this.observe(value);
    const dep = new Dep();
    Object.defineProperty(data,key,{
      enumerable: true,
      configurable: false,
      get(){
        //每次取值时调用
        //订阅数据变化时，往dep添加观察者
        Dep.target && dep.addSub(Dep.target)
        return value
      },
      set:(newVal)=>{
        this.observe(newVal)
        if(newVal !== value ){
          value = newVal;
          //告诉 dep 通知变化
          dep.notify();
        }
      }
    })
  }
}
```

index.html

```html
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>write vue</title>
</head>
<body>
  <div id="app">
    <input type="button" v-on:click='xxxs' value="button-on"/>
    <input type="button" @click='xxxs' value="button-@"/>
    <a v-bind:href="url">bind url 百度</a>
    <p v-bind:class="bindClass">bind class red</p>
    <div type="text" v-html='msg'></div>
    <p type="text" v-text='person.name'></p>
    <input type="text" v-model='msg'/>
    {{person.name}}-{{person.age}}
  </div>
  <script src="./mvue.js"></script>
  <script src="./observer.js"></script>
  <script>
    let vm = new mvue({
      el:'#app',
      data:{
        person:{
          name:'111',
          age:'26'
        },
        msg:'model dire',
        msg2:'<a href="javascript:void(0);">html指令</a>',
        msg3:'sba',
        url:'http://www.baidu.com',
        bindClass:{
          color:'red',
          display:'block'
        }
      },
      methods: {
        xxxs(){
          this.person.name = '222'
          this.msg2 = '<a href="javascript:void(0);">html指令2</a>'
          this.msg = 'model dire222'
        }
      }
    })
  </script>
</body>
</html>

```
