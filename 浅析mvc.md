 # 1.MVC 三个对象
mvc把代码分成三块：

* M 是 Model，数据模型，负责数据相关的任务，数据相关代码放到 m
* V 是 View，视图，负责用户界面，视图相关代码 v
* C 是 Controller，控制器，负责监听用户事件，然后调用 M 和 V 更新数据和视图，其他相关代码放到 c
```
import "./app1.css"  
import $ from 'jquery'

//数据相关 m
const m ={
  data:{
    n : parseInt(localStorage.getItem("n")) 
  },
  create(){},
  delete(){},
  update(data){
    Object.assign(m.data,data)
    eventBus.trigger('m:updated')
  },
  get(){}
}
//视图相关 v
const v ={
  el:null,
  html:`
<div>
  <div class="output">
      <span id="number">{{n}}</span>
  </div>
  <div class="actions">
      <button id="add1">+1</button>
      <button id="minus1">-1</button>
      <button id="mul2">×2</button>
      <button id="divide2">÷2</button>
  </div>
</div>      
`,
init(container){
  v.el = $(container)
},
render(){
  if(v.el.children.length != 0)
   v.el.empty()
   $(v.html.replace('{{n}}',m.data.n))
    .appendTo(v.el)
  }
}
//其他都放到 c
const c = {
  init(container){
    v.init(container),
    v.render(m.data.n)
    c.autoBindEvents()
    eventBus.on('m:updated',()=>{
      v.render(m.data.n)
    })
  },
  //表驱动编程
  events:{
    'click #add1':'add',
    'click #minus1':'minus',
    'click #mul2':'mul',
    'click #divide2':'divide'
  },
    add(){
    m.update({n:m.data.n+=1}) 
    },
    minus(){
    m.update({n:m.data.n-=1}) 
    },
    mul(){
    m.update({n:m.data.n*=2}) 
    },
    divide(){
    m.update({n:m.data.n/=2}) 
    },
  autoBindEvents(){
   for(let key in  c.events){
    const value =c[c.events[key]]
    const spaceIndex =key.indexOf(' ') 
    const part1 =key.slice(0,spaceIndex)
    const part2 =key.slice(spaceIndex + 1)
    v.el.on(part1,part2,value)
   }
  }
}
export default c
```
# 2.EventBus 的API
eventBus 主要用于对象间通信。
EventBus的API很多，常件的有以下几个：
* addClass:ƒ( value )
* removeClass:ƒ(  value )
* toggleClass:ƒ( value, stateVal )
* appendTo:ƒ( selector )
* bind:ƒ( types, data, fn )
* click:ƒ( data, fn )
* on:ƒ( types, selector, data, fn )
* off:ƒ( types, selector, fn )
* trigger:ƒ( type, data )等。。。
以下代码是触发事件eventBus.trigger()和监听事件eventBus.on()的示例：

eventBus 满足了最小知识原则，m 和 c 互相不知道对方的细节，但是却可以调用对方的功能.
```
//eventBus对象间通信，有on事件和trigger事件
const eventBus = $({})
console.log(eventBus)打印出eventBus的API
//数据相关 m
const m ={
  data:{
    n : parseInt(localStorage.getItem("n")) 
  },
  create(){},
  delete(){},
  update(data){
    Object.assign(m.data,data)
    eventBus.trigger('m:updated')  //eventBus触发事件 API eventBus.trigger()
  },
  get(){}
}
//其他都放到 c
const c = {
  init(container){
    v.init(container),
    v.render(m.data.n)
    c.autoBindEvents()
    eventBus.on('m:updated',()=>{   //eventBus点击事件 API eventBus.on()
      v.render(m.data.n)
    })
  },
  //表驱动编程
  events:{
    'click #add1':'add',
    'click #minus1':'minus',
    'click #mul2':'mul',
    'click #divide2':'divide'
  },
    add(){
    m.update({n:m.data.n+=1})        //
    },
    minus(){
    m.update({n:m.data.n-=1})        //
    },
    mul(){
    m.update({n:m.data.n*=2})        //
    },
    divide(){
    m.update({n:m.data.n/=2})        //
    },
  autoBindEvents(){
   for(let key in  c.events){
    const value =c[c.events[key]]
    const spaceIndex =key.indexOf(' ') 
    const part1 =key.slice(0,spaceIndex)
    const part2 =key.slice(spaceIndex + 1)
    v.el.on(part1,part2,value)
   }
  }
}
```
# 3.表驱动编程
表指的是哈希表，表驱动编程是一种编程模式,适用场景:消除代码中频繁的if else或switch case的逻辑结构代码,使代码更加直白，减少重复代码，只讲重要的信息放在表里，然后利用表来编程
.
```
//表驱动编程
  events:{
    'click #add1':'add',
    'click #minus1':'minus',
    'click #mul2':'mul',
    'click #divide2':'divide'
  },
    add(){
    m.update({n:m.data.n+=1})        //
    },
    minus(){
    m.update({n:m.data.n-=1})        //
    },
    mul(){
    m.update({n:m.data.n*=2})        //
    },
    divide(){
    m.update({n:m.data.n/=2})        //
    },
 ```
# 4.模块化编程
由于一个页面写的js太多了，都写在一个文件中，不好调试。所以就需要把js文件，分割成易于维护的代码块，之间能相互调用方法和属性。
但是需要处理好模块间的依赖关系。
## 一、原始写法

模块就是实现特定功能的一组方法。

只要把不同的函数（以及记录状态的变量）简单地放在一起，就算是一个模块。
```
　　function m1(){
　　　　//...
　　}

　　function m2(){
　　　　//...
　　}
```
上面的函数m1()和m2()，组成一个模块。使用的时候，直接调用就行了。

这种做法的缺点很明显："污染"了全局变量，无法保证不与其他模块发生变量名冲突，而且模块成员之间看不出直接关系。

## 二、对象写法

为了解决上面的缺点，可以把模块写成一个对象，所有的模块成员都放到这个对象里面。
```
　　var module1 = new Object({

　　　　_count : 0,

　　　　m1 : function (){
　　　　　　//...
　　　　},

　　　　m2 : function (){
　　　　　　//...
　　　　}

　　});
```
上面的函数m1()和m2(），都封装在module1对象里。使用的时候，就是调用这个对象的属性。

　　module1.m1();

但是，这样的写法会暴露所有模块成员，内部状态可以被外部改写。比如，外部代码可以直接改变内部计数器的值。

　　module1._count = 5;

## 三、立即执行函数写法

使用"立即执行函数"（Immediately-Invoked Function Expression，IIFE），可以达到不暴露私有成员的目的。
```
　　var module1 = (function(){

　　　　var _count = 0;

　　　　var m1 = function(){
　　　　　　//...
　　　　};

　　　　var m2 = function(){
　　　　　　//...
　　　　};

　　　　return {
　　　　　　m1 : m1,
　　　　　　m2 : m2
　　　　};

　　})();
```
使用上面的写法，外部代码无法读取内部的_count变量。

　　console.info(module1._count); //undefined

module1就是Javascript模块的基本写法。下面，再对这种写法进行加工。

## 四、放大模式

如果一个模块很大，必须分成几个部分，或者一个模块需要继承另一个模块，这时就有必要采用"放大模式"（augmentation）。
```
　　var module1 = (function (mod){

　　　　mod.m3 = function () {
　　　　　　//...
　　　　};

　　　　return mod;

　　})(module1);
```
上面的代码为module1模块添加了一个新方法m3()，然后返回新的module1模块。

## 五、宽放大模式（Loose augmentation）

在浏览器环境中，模块的各个部分通常都是从网上获取的，有时无法知道哪个部分会先加载。如果采用上一节的写法，第一个执行的部分有可能加载一个不存在空对象，这时就要采用"宽放大模式"。
```
　　var module1 = ( function (mod){

　　　　//...

　　　　return mod;

　　})(window.module1 || {});
```
与"放大模式"相比，＂宽放大模式＂就是"立即执行函数"的参数可以是空对象。

## 六、输入全局变量

独立性是模块的重要特点，模块内部最好不与程序的其他部分直接交互。

为了在模块内部调用全局变量，必须显式地将其他变量输入模块。
```
　　var module1 = (function ($, YAHOO) {

　　　　//...

　　})(jQuery, YAHOO);
```
参考链接：http://www.ruanyifeng.com/blog/2012/10/javascript_module.html