---
title: Javascript语言之蜜汁this
date: 2017-03-27 01:51:38
categories: 
- javascript 
tags: 
- this
- javascript核心
---

this关键字是Javascript中最重要的机制之一。关于介绍他的文章也比比皆是，而大多都浅尝辄止，浮于表面,不够系统。故整理此篇。
### 先看几个题目
题目1 //默认绑定  严格模式 非严格模式

``` javascript
    function foo(){
      console.log(this.a);
    }
    var a = '2';
    (function(){
      'use strict';
      foo(); 
    })()
```
<!-- more -->
        

题目2 //隐式绑定
``` javascript
    function foo(){
        console.log(this.a);
    }
    var a = 1;
    var obj2 = {
        a: 2,
        foo:foo
    }
    var obj1 = {
        a: 1,
        obj2: obj2
    }
    
    obj1.obj2.foo() 
```
题目3 
   
``` javascript
    function foo(){
        console.log(this.a);
    }
    
    var a = 2;
    var o = {a:3,foo:foo};
    var p = {a:4};
    
    (p.foo = o.foo)() 
    
    //知识点：赋值表达式的返回值
```
题目4 
``` javascript
    var obj1 = {
        a: 1,
        foo: function(){
          console.log(this.a);
        }
    };
    var obj2 = { a: 2 };
    var obj3 = { a: 3};
    obj1.foo.bind(obj2).call(obj3) 
```
    //知识点： bind实现
  
  上面4个题目的答案都是：2 。<br/>
  如果你都答对了，那可以继续往下看了,以下是正文。本文先说为什么要用this，再说怎么用this。
    
### 为什么要用this?


先看以下这段简单的代码
``` javascript
    function speak() {
        var greeting = "Hello, I'm " + this.name;
        console.log( greeting );
    }
    
    var me = {
        name: "小a"
    };
    
    var you = {
        name: "小b"
    };
    
    speak.call( me ); // Hello, I'm 小a
    speak.call( you ); // Hello, I'm 小b
```
* 以上代码可以在不同上下文对象（me 和 you）中重复使用speak函数。
* 如果不使用this,就要给 speak显示传入一个上下文对象（如下）
        
``` javascript
        function speak(content) {
            var greeting = "Hello, I'm " + content.name;
            console.log( greeting );
        }
      
        speak(me);  // Hello, I'm 小a
        speak(you); // Hello, I'm 小b
```
显然，this提供了一种更优雅的方式来隐式传递一个对象的引用，因此可以使API设计得更加简洁且易于复用。   
随着使用的模式越来越复杂，显示传递上下文对象会让代码变得越来越混乱，使用this则不会这样。
           
### this是什么？

说起this,这要看执行上下文是什么。javascript中根据可执行的种类分全局执行上下文和函数执行上下文（ES6有了块级上下文）。（关于执行上下文见另一篇[博文](https://foreverwang.github.io/2017/04/09/Javascript%E4%B9%8B%E6%89%A7%E8%A1%8C%E4%B8%8A%E4%B8%8B%E6%96%87/)）
在全局执行上下文中（在任何函数体外部）很简单，this指向全局对象（不管是否是严格模式下）。以下说的都是在函数上下文中的this。
#### this常见误区
* this 不指向函数自身（就没必要附加例子了）
* this 任何情况下也不指向函数的词法作用域（es6 箭头函数 本身没有this，那是他外部作用域的this）        
``` javascript
        function foo(){
            var a = 2;
            this.bar();
        }
        function bar(){
            console.log(this.a);
        }
        
        foo()//undefined
```
    这段代码试图通过this联通 foo和bar的词法作用域，从而让bar可以访问foo作用域里的变量a,当然这里是不能如愿的。不能使用this来引用一个词法作用域内部的东西。

 



#### this到底是啥
 this 是在运行时进行绑定的，并不是在编写时绑定，他的上下文取决于函数调用时的各种条件。<br/> （this的值只和函数调用有关，和函数定义无关---这个说法是不对的，有两个例外：bind 和箭头函数，后面说）
 当一个函数被调用的时候，进入这个函数的执行上下文。执行上下文里包含：函数的活动对象（函数传入的参数等）；作用域链 （函数调用栈等）；this就是上下文的其中一个属性，会在函数执行过程中用到。

### 函数作用域内this绑定规则（4个）
先确定函数的调用位置，调用栈
#### 1.默认绑定
先说结论：

* 什么是默认绑定： 非严格模式下，this指向全局对象；严格模式下，this为undefined
* 何时会走默认绑定：独立函数调用

函数在不带任何修饰的进行调用（即独立函数调用）的时候，this会走默认绑定规则。
<br>非严格模式下：this指向全局对象
``` javascript
        function foo(){
            console.log(this.a);
        }
        var a = 2;
        foo();//2 
```
严格模式下：this为undefined
``` javascript
     function foo(){
         'use strict';
          console.log(this.a);
     }
     var a = 2;
     foo()//Cannot read property 'a' of undefined
```
这里要注意一点， 对于默认绑定，决定this绑定对象的不是调用位置是否处于严格模式，而是函数体是否处于严格模式，这也是题目1中 看似是在严格模式下，this却绑定到了全局对象
``` javascript
    function foo(){
      //foo函数体处于非严格模式
      console.log(this.a);
    }
    var a = '2';
    (function(){
      'use strict';
      //该立即执行的函数表达式内处于严格模式这里的this为undefined
      console.log(this);//undefined
      foo(); //2
    })()  
```
#### 2.隐式绑定
先说结论:<br>
当函数被调用时，函数引用有上下文对象，隐式绑定规则会把函数调用中的this绑定到这个上下文对象
再看题目2<br>
``` javascript
    function foo(){
        console.log(this.a);
    }
    var a = 1;
    var obj2 = {
        a: 2,
        foo:foo
    }
    var obj1 = {
        a: 1,
        obj2: obj2
    }
    
    obj1.obj2.foo() //2
    //对象属性引用链中只有最顶层(属性的直接调用方)会影响调用位置
```
<br>
   隐式丢失（严格来说此处并没有绑定过，也没有丢失一说，只是看上去像是丢失了）
``` javascript
    function foo(){
        console.log(this.a);
    }
    var obj = {
        a: 1,
       foo: foo
    }
  
    var bar = obj.foo;//函数别名
    var a = 2;
    bar();
```
  虽然bar是obj.foo的一个引用，实际上，他引用的是foo本身，此时的bar调用 就是一个不带任何修饰的函数调用，因此应用了默认绑定。<br>
 其实本例有两个关键点：
 
  * 赋值表达式的返回值（犀牛书6版81页）： var bar = obj.foo;返回的是右值，右值指向foo的引用
  * 不管如何引用，只看调用时 调用是bar() ---> foo()

此时再看题目3，就很清晰了
``` javascript
    function foo(){
        console.log(this.a);
    }
    
    var a = 2;
    var o = {a:3,foo:foo};
    var p = {a:4};
    
    //o.foo();//3
    (p.foo = o.foo)() //p.foo = o.foo的返回值是等号的右值-->foo的引用，此处相当于直接调用foo()
    //p.foo()//4
```

#### 3.显示绑定 

先说结论：

* 函数调用时通过 apply 或者 call 硬绑定this 对象(apply和call的区别这里就不赘述了)
* 通过ES5的 Function.prototype.bind 返回一个绑定了this了的新函数

看一个硬绑定的典型应用场景:接受不确定参数
``` javascript
    function foo(sth){
        console.log( this.a,sth );
        return this.a + sth;
    }
    var obj = {
        a: 1
    }

    var bar = function(){
        return foo.apply(obj,arguments);
    }
    
    var b = bar(1);
    console.log(b);//  2  
```
另一种方法是创建一个可复用的辅助函数（bind）
``` javascript
    function foo(sth){
        console.log( this.a,sth );
        return this.a + sth;
    }
    
    //简单的辅助绑定函数
    function bind(fn,obj){
        return function(){
            return fn.apply(obj,arguments);
        }
    }
    
     var obj = {
        a: 1
    }
    
    var bar = bind(foo,obj);
    var b = bar(1);
    console.log(b);//2
```

   由于硬绑定是一种很常用的模式，ES5提供了内置的Function.prototype.bind方法：<br>
   `bind方法`(犀牛书第6版190页)将返回一个新的函数，以函数调用的方式调用新的函数将会把原始函数当做bind的第一个参数的方法来调用，传入新函数的任何实参都将传入原始函数。根据定义，用ES3很容易模拟bind方法，即上面代码中我们自定义的bind函数（这里只是把最基本的绑定this模拟了，参数柯里化等没有模拟，可以看MDN bind的polyfill写法，这里不贴了）。
  <br>
此时，再看题目4应该就很清晰了：
``` javascript
    var obj1 = {
        a: 1,
        foo: function(){
          console.log(this.a);
        }
    };
    var obj2 = { a: 2 };
    var obj3 = { a: 3};
    obj1.foo.bind(obj2).call(obj3)    
```
    
  最后一行中 obj1.foo.bind(obj2)  等价于以下
``` javascript
    function bar(){
        obj1.foo.apply(obj2,arguments)
    }
```
   所以foo.bind(obj2)后返回了一个新的函数，这个函数被调用的时候的this被绑定到obj2，再call(obj3) 也是改变不了这个事实。
 
   bind方法和this的关系就是以上了，由于bind方法实在是很重要并且很好用，所以这里把bind方法说完。
   <br>
   
   bind方法不仅是把函数绑定至一个对象，他还附带其他功能： 
   <br>
   
 * bind()的另一个最简单的用法是使一个函数拥有`预设的初始参数`。
 这些参数（如果有的话）作为bind()的第二个参数开始跟在this（或其他对象）后面，之后它们会被插入到目标函数的参数列表的开始位置，调用绑定函数时传递给绑定函数的参数会跟在它们的后面。这个附带的应用即是函数式编程中的"柯里化"(currying)的一种（todo:关于柯里化以后再深入研究下）。

``` javascript
        function foo(y,z){
            return this.x + y + z;
        }
        
        var baz = fo.bind({x:1},2);
        baz(3);
```
   
 bind方法返回一个函数对象我们叫`绑定函数`，绑定函数的length属性值是被调函数（绑定函数的目标函数）的形参个数减去绑定实参个数（length值不能小于0）。
        
 * bind返回的函数被`用作构造函数`，将忽略传入bind的this,原始函数就会以构造函数的形式调用。bind时传入的实参会原封不动的传入原始函数，调用绑定函数是传入的参数紧跟其后。      
        
   
#### 4.new 绑定 
先看下new 一个构造函数都发生了什么：

* 新建一个新对象（继承自Constructor.prototype）
* 将构造函数的作用域赋给新对象，因此this指向这个新对象
* 执行构造函数内的代码（为这个对象添加属性）
* 隐式返回这个对象（如果构造函数内没有显示return 非null对象值）
   
很清晰，new 的时候 this绑定到 创建的新对象。通常是返回的那个对象实例。
多说一句，起始在js中没有构造函数一说，只有构造调用一说。   
   


#### 绑定优先级   
四种绑定规则说完了，那么问题来了，如果有多重规则同时作用时，优先级是怎样的呢？
先回顾一下四种规则：

* 默认绑定（严格模式、非严格模式）
* 隐式绑定（对象属性调用）
* 显示绑定（apply/call、bind）
* new 绑定


直接说结论吧 <br>
new 绑定 > 显示绑定 > 隐士绑定 
<br>在没有前三种绑定规则的时候就应用默认绑定。

至此，貌似事件处理函数中的 this还没说。事件处理函数又分DOM2级事件处理函数（addEventListener）;和内联事件处理函数。前者指向绑定事件的DOM可以归到显示绑定吧，只是这一步引擎帮你做了。后者this指向全局对象属于默认绑定，不再赘述。

### 绑定例外
在规则的世界里一切秩序井然，然而世事总有些例外，this绑定也不例外。

#### 被忽略的this

当把 null或者undefined作为this的绑定对象传入 call,apply,bind,这些值在调用时会被忽略，通常实际应用的是默认绑定规则。
``` javascript
    function foo(){
      console.log(this.a);
    }
    var a = 2;
    foo.call(null);//2
```
既然有这个机制，通常他就应该是有用的。那什么时候我们会用到null 、nudefined值呢?
当你只想用来传递参数而不关心this的话，这是个不错的选择


``` javascript
    //使用apply(...)来展开一个数组，并当做参数传入一个函数
    function foo(a,b){
        console.log('a:'+a,'b:'+b);
    }
    //当然，ES6中我们可以用 ...操作符来代替apply展开数组了
    
    foo.apply(null,[2,3]) //a:2,b:3
    
    //使用bind(...)对参数进行柯里化（预设一些参数）
    var bar = foo.bind(null,2);
    bar(3);//a:2,b:3
```
至此，this讲解就接近尾声了--还有关于this的最后一点：this词法。
    
### this词法
 我们知道javascript使用了词法作用域，但javascript的this机制某种程度上很像动态作用域，因为this的值通常跟函数调用有关，而跟词法作用域无关。<br>
 事情一直到ES6之前是这样的。ES6箭头函数使得this的值和词法作用域联系起来。
 
 * 箭头函数里this值不实用前面的任何规则，而是定义该箭头函数式执行上下文里this的值。
 * 箭头函数的this绑定无法被修改 (new 也不行)


又到了举个栗子的时候了：
``` javascript
		function foo(){
			return (a) => {
				//this继承自foo
				console.log(this.a);
			}
		}
		
		var obj1 = {a:2};
		var obj2 = {a:3};
		var bar = foo.call(obj1);
	   bar.call(obj2);//2
```
感觉这个箭头函数里的this值似曾相识？是的，在ES6之前，我们就经常使用一种和箭头函数一样的模式。
``` javascript
	function foo(){
		var self = this; 
		setTimeout(function(){
			console.log(self.a);
		},100);
	}	
	
	var obj = {a:2};
	foo.call(obj);//2
```
（完）

关于this的细节就讲完了。this作为执行上下文的一个属性，由于他的规则比较多所以单独拿出来说。关于执行上下文的其他内容甚至更为重要，接下来建议拓展阅读另一篇博文[javascript之执行上下文](https://foreverwang.github.io/2017/04/09/Javascript%E4%B9%8B%E6%89%A7%E8%A1%8C%E4%B8%8A%E4%B8%8B%E6%96%87/)。
私以为搞透执行上下文，是真正理解javascript这门语言的必要条件。
	
	
	


### 参考资料
* [《你不知道的javascipt》](https://github.com/getify/You-Dont-Know-JS/tree/master/this%20%26%20object%20prototypes)
* 《javascript权威指南》
