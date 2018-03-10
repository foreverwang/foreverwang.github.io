---
title: Javascript之执行上下文
date: 2017-04-09 16:21:50
categories: 
- javascript 
tags: 
- 作用域链
- 执行期上下文
- javascript核心

---

### 关键词
* 执行期上下文（excution context简称EC）：简称上下文（又叫执行环境）
* 作用域链 （scope chain）
* 变量对象（variable object简称VO）
* 活动对象（activation object简称AO）

### 概述
执行上下文是Javascript中最重要的一个概念。每一段代码的执行都与它息息相关。理解了它，才能真正理解我们写的的javascript代码是如何运行的。
ECMA-262（5.1）中写道：当控制器转入 ECMAScript的可执行代码时，控制器会进入(注：创建)一个执行上下文。
<!-- more -->
#### 可执行代码
这里先解释下上面提到的可执行代码，我们看一下ES5规范（以下从简了）

* 全局代码：是指被作为ECMAScript程序处理的源代码文本。一个特定程序的全局代码不包括作为函数体被解析的源代码文本。
* 函数代码：是指作为函数体被解析的源代码文本。不包括作为其嵌套函数的 函数体 被解析的源代码文本。
* Eval 代码 是指提供给eval内置函数的源代码文本。（现在应用比较少，这一点本文忽略）


现在可以对上面关于上下文的说法换一种说法：<br>

* 当控制器转入 ECMAScript的全局代码，控制器会创建一个全局上下文。
* 当转入函数代码，控制器会创建一个函数上下文。

### 执行上下文
我们已经知道了什么时候会创建一个上下文，那接下来就要说下上下文具体是个什么东西。<br>
ES5规范里说：执行上下文包含所有用于追踪与其相关的代码的执行进度的状态。<br>
翻译一下是说：执行上下文里有代码执行时需要用到的东西。<br>
现在问题就具体到了：执行上下文里都有什么？那么我们来看一下：

#### 执行上下文的组成

| 组件|内部组成 |作用|
|--- |--- | ---|
|变量对象（VO） | {vars:...,function declarations:...,arguments:...,...}| 指定一个词法环境对象，其环境数据用于保存由该执行上下文内的变量声明 和 函数声明。|
|作用域链（scope chain）     | [ variable object + [[scope]]）  |指定一个词法环境对象，用于解析该执行环境内的代码创建的标识符引用。|
|this指针|  |指定该执行上下文内的 ECMAScript代码中this关键字所关联的值。 |



为了方便理解，一个执行上下文可以抽象为object。每一个执行上下文都有一系列的属性（我们称为上下文状态，即上面表格做左侧组件一列的属性）。接下来就对上面表格中的三个状态属性来一一说明。

#### 变量对象  （VO）

变量对象是与执行上下文相关的数据作用域(scope of data) ,用于存储被定义在上下文中的变量声明和函数声明(注意：不包括函数表达式[详见](http://www.cnblogs.com/TomXu/archive/2011/12/29/2290308.html)) 。<br>
定义已经很明确了，我们通过具体的栗子看下全局上下文的VO：

``` javascript
    var foo = 2;        
    function bar() {} // 函数声明
    (function baz() {}); // 函数表达式   
    console.log(
      this.foo == foo, // true
      window.bar == bar // true
    );  
    console.log(baz); // Uncaught ReferenceError: baz is not defined
```
该例中全局上下文中变量对象会有以下属性：    
    
|global VO||
|---|---|   
|foo | undefined --> 10|
|bar | function... |    
|built-ins（忽略）| 一些内置的全局变量

这是全局上下文中的变量对象，除了全局上下文还有函数上下文，那么函数上下文是否是一样的呢？答案是稍微有点不一样：函数内活动对象（AO）用作变量对象。接下来我们看下AO。<br>

>在global全局上下文中，变量对象也是全局对象自身[global object],如浏览器中是window,此时我们可以通过全局对象的属性来指向全局变量,通过this来访问全局对象。然而在函数上下文内我们是无法直接访问这个对象的,this也不指向这个对象。<br>
    
#### 函数的活动对象（AO）
    
当函数被调用者激活（即函数调用），活动对象 就被创建了。它包含普通参数即形参(formal parameters) 与特殊参数(arguments)对象(具有索引属性的参数映射表)。活动对象在函数上下文中作为变量对象使用。
    
   还是举个栗子看下函数的活动对象:
``` javascript
    function foo(x, y) {
      console.log(z); //undefined
      var z = 3;
      console.log(z);//3
      function bar() {} // 函数声明
      (function baz() {}); // 函数表达式
    }
    foo(1, 2);
```
   上面当foo被调用时，foo函数上下文的活动对象(AO)被创建，其内容如下表格：
  
  
|Activation object| |
| ---|---|
|x | 1 |
|y | 2 |
|arguments| { {0:{x:1}, {1:{y:1}} }|
|z | undefined-->3|
|bar| funciton |
   
   
前面我们已经知道变量对象中不包含函数表达式，这里活动对象内也不包括函数表达式。

我们已经知道了VO和AO的内容组成了，那么他们是什么时候被创建的，又是什么时候改变的呢？<br>
>VO|AO在每次创建上下文时作为上下文的组成部分被创建，并填入初始值;值的更新出现在代码执行阶段。<br>


上面两个例子中我们知道了，全局上下文中，我们定义的变量和声明的函数都作为全局上下文的变量对象的属性来保存，代码执行时标志符的查找也是从全局上下文的变量对象中查找。在某一个函数上下文中也类似，在当前上下文中的标志符会先在当前变量对象（函数上下文中是活动对象）中查找。但是当在当前上下文的变量对象找不到的时候会发生什么呢？答案是：完整的标志符解析是通过作用域链（scope chain）机制来完成的。（不容易啊，终于比较自然的把作用域链给引出来了）。那么我们来看一下什么是作用域链。

#### 作用域链（scope chain）

作用域链是一个 对象列表(list of objects) ，用以检索上下文代码中出现的 标识符(identifiers) 。先说结果：

> scope chain = VO|AO + [[scope]] 

依然看个栗子：
``` javascript
    var x = 1;
    function foo() { 
      var y = 2; 
      function bar() {
        console.log(x + y);
      } 
      return bar; 
    }
     
    foo()(); // 3
```

这段代码执行完，共创建了三个执行上下文：执行流进入这段全局代码会创建一个全局上下文，foo调用时创建foo上下文，bar被调用时创建bar上下文(foo()-->bar调用)。
这里我们暂且只看一下bar的上下文，我们已经知道一个上下文的数据域即一个上下文的变量对象,在函数上下文即是活动对象（AO），那我们就看下bar函数的活动对象。<br>
当执行流进入bar函数，且函数体内代码执行之前，javascript引擎会创建这个函数的AO：

| AO||
|---|---|
|arguments|[callee: function, Symbol(Symbol.iterator): function]|

我们看到bar函数提内代码执行之前创建的这个AO并没有函数体中需要用到的变量x和y,那么当代码执行到console.log(x + y)时，x和y是从哪里读取的呢？不难看出：y存在创建的foo函数上下文的活动对象中；x存在创建的全局上下文的变量对象中。好像是函数内可以访问到函数祖父级上下文的变量对象里的东西。实际上也是这样的。而能访问到祖父上下文的变量对象，正是通过函数的一个内部属性--[[scope]]实现的。

##### [[scope]]
[[scope]]是ECMA262规定的对象的私有属性，理论上只有JS引擎可以访问。

* [[scope]]是所有祖父变量对象的层级链。
* [[scope]]在函数创建时被存储－－静态（不变的），直至函数销毁。


拓展阅读[例子](http://www.cnblogs.com/TomXu/archive/2012/01/18/2312463.html)。

所以一个函数内代码执行时，函数内遇到的标志符就是这样先从当前上下文的活动对象内查找，若找不着继续查找父上下文的变量对象，直到查到全局上下文。而这也正是javascript中的`变量标志符查找机制`。


##### 标志符解析机制
我们直接看规范（ES3 10.1.4章节）：

每个执行上下文都有一个与之相关联的作用域链（scope chain）。作用域链是一个由对象组成的链表，求值标志符的时候会对它搜索。当创建一个上下文时，根据当前上下文的代码类型（全局代码或函数代码）一个作用域链被创建，并用初始化对象填充（函数代码被填充为函数的[[scope]]属性值-接下来讲；全局代码初始化为空吧。当VO|AO被创建后，VO|AO被推到作用域链的前端）。一个上下文中代码执行时，其作用域链只会被 with 声明（见 12.10）和 catch 语句（见 12.14）所影响。标志符解析的具体规则：

1. 获取作用域链中的下一个对象。如果没有，转到步骤 5。
2. 调用结果(1) 的 [[HasProperty]] 方法，把标识符（Identifier）作为属性名传递。
3. 如果结果(2) 为true，返回一个引用类型的值，其基对象（base objecqit）是结果(1),其property name是该Identifier。
符。
4. 转到步骤 1。
5. 返回一个引用类型的值，其base object为 null，其property name 是Identifier。
求值标识符的结果总是一个引用类型的值，其成员名字组件与标识符字符串相等。

> 注：Reference(引用)类型的值是JS引擎使用的一种数据类型，它分为base object和property  name两个部分。假设在JS代码中有obj.prop这样的表达式，那么解释成Reference类型，base object是对象obj,而property name是字符串”prop”。--winter
  

#### this
至此上下文里还有this没说，我的总结是：this通常指向激活当前上下文的那个对象。this值在进入上下文时确定，并且在上下文运行期间不能被改变。<br> 
``` javascript
    console.log(this);// global object
    var  obj = { a: function(){ 
                        console.log(this)
                    }
                  }; 
     obj.a();//obj 可以理解为：obj.a的上下文是被obj这个对象激活的。
```
当然this绑定还有一些其他规则。详见另一篇博文[javascript之蜜汁this](https://foreverwang.github.io/2017/03/27/Javascript%E8%AF%AD%E8%A8%80%E4%B9%8B%E8%9C%9C%E6%B1%81this/)。


#### 总结一下上下文：<br>
上下文分为全局上下文和函数上下文。<br>
全局上下文没啥好总结的，我们就看在函数上下文的整个生命周期。

执行流进入一个函数时，引擎不是简单的立即执行函数体内的代码，可分为两个阶段：

* 第一阶段：上下文创建阶段
	* 一个函数上下文被创建，同时作为上下文的一部分的作用域链也被创建，并被初始化为函数[[scope]]属性的值。
	* 接着javascript引擎创建当前上下文的活动对象（AO）,并且将AO推入作用域链的最前端。
	* 确定this的值。
	
* 第二阶段：代码逐行执行--变量赋值、函数引用等	。

#### 上下文栈（EC）
以上部分我们对单个上下文的创建和组成做了详细的说明。而一段代码的执行往往是涉及到很多个上下文。而在代码执行过程中这些上下文之间是什么关系呢？ <br>
活动的上下文在逻辑上组成一个上下文栈。栈底是全局上下文，而栈顶是当前激活的上下文。<br>
当js引擎执行全局代码前，会首先创建一个全局上下文。全局上下文创建完毕后，全局代码开始逐行被执行。代码执行过程中当一个函数被调用时，此时引擎会创建一个函数上下文，并且将其`推入`到上下文栈顶。引擎总是执行当前在栈顶的上下文的代码，函数执行完毕，上下文栈将该函数上下文`弹出`，控制权返回给之前的上下文。ECMAScript程序中的执行流正式被这个机制控制者。

##### 相关推荐：


* [大叔的深入理解JavaScript系列（10）：JavaScript核心](http://www.cnblogs.com/TomXu/archive/2012/01/12/2308594.html)
* [What is the Execution Context & Stack in JavaScript?](http://davidshariff.com/blog/what-is-the-execution-context-in-javascript/) [(译文)](http://yanhaijing.com/javascript/2014/04/29/what-is-the-execution-context-in-javascript/)
*  [ winter的JavaScript中的\[\[scope\]\]和Scope Chain](http://www.cnblogs.com/winter-cn/archive/2008/07/07/1237168.html)

##### 结语
本文主要参考了[ES3规范的第10章节Execution Contexts](http://www.ecma-international.org/publications/files/ECMA-ST-ARCH/ECMA-262,%203rd%20edition,%20December%201999.pdf)。行文上部分参考上面两篇博文。写的过程中，发现把这个主题涉及的内容很连贯的串起来还是不容易的。而网上的关于该主题的博文也没有（我没找到）很好的把这些内容连贯的串起来的。所以整理此文。不管怎样，至少自己梳理来了一边，更清晰了。
执行上下文大概就这些东西了吧。这大概也是整个javascript这门语言最核心且没有之一的东西了。







    
    
    








   
  

   
   
   
   
   
   
   
   


   



























