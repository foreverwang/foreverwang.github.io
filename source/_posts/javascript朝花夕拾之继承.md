---
title: javascript朝花夕拾之继承
date: 2018-04-05 22:16:08
tags:
 - javascript
 - 继承
categories: 
 - javascript
---

从实现角度，在OO语言中的继承特性，有三种实现方案，包括基于类（class-based）、基于原型（prototype-based）和基于元类（metaclass-based）。

从形式上，许多OO语言都支持两种类型继承方式：接口继承和实现继承。接口继承只继承方法签名，实现继承则继承实际的方法。js 中函数没有签名，故js 中无法实现接口继承。js 中只支持实现继承，而其实现继承主要是依靠原型链来实现的。


### 构造函数，原型对象，实例对象的关系

每一个构造函数都有一个`prototype`属性指向其原型对象；每一个原型对象都包含一个指向构造函数的指针（或者说属性）`constructor`;而实例对象都包含一个指向原型对象的内部指针(内部属性) `[[prototype]]`。

对象实例是无法直接通过实例属性来访问原型对象的，但是我们可以间接方式显式的访问其原型对象。 通过在实例上访问其constructor 属性可以引用到实例的构造函数，再访问构造函数的prototype属性访问原型对象。

<!-- more -->

代码示例：


    function A() {}
    var instance = new A;	


通过instance访问原型对象： instance.constructor.prototype 。
对象实例内部属性[[prototype]]对程序而言是不可见的，即在代码里不能直接访问，供引擎来访问。 但在浏览器实现中每个对象都有一个` __proto__` 属性，可以在代码层面直接访问原型对象。


### 基于原型链的继承
上面我们理解了原型对象和构造函数及实例对象的关系。 如果我们让原型对象等于另一个类型的实例。 此时原型对象就包含一个指向另一个原型对象的指针，相应的另一个原型对象也包含一个指向两一个构造函数的指针。假如另一个原型对象又是另一个类型的对象的实例，那么上述关系依然存在，如此层层递进,就构成了实例和原型的链条，这就是`原型链`的概念。

> 实现原型链的基本模式，让一个原型对象等于一个对象的实例。
代码如下：

``` javascript

    function SuperType() {
        this.property = true;
    }
    
    SuperType.prototype.getSuperValue = function() {
    	return this.property;
	 };
    
    function SubType() {
    	this.subProperty = false;
    }
    
    SubType.prototype = new SuperType();
    
    SubType.prototype.getSubValue = function() {
    	ruturn this.subProperty;
    }
    
    var instance = new SubType();
	 console.log(instance.getSuperValue()); //true
	 
 ```   


引用关系图如下:
    
![继承关系图](https://ws1.sinaimg.cn/large/d16dcf79gy1fpxkbh9blaj20ve0eqmz9.jpg)


### 原型链的问题
原型链虽然很强大，可以用它实现继承有两个明显的问题。

* 1.但当原型对象中包含引用类型值时，会有问题。 由于原型对象的属性被所有实例共享，当一个实例去修改一个共享自原型上的引用类型的属性时，会影响到所有其他实例。示例代码如下：

``` javascript
	
	function SuperType() {
	    this.colors = ["red", "blue", "green"];
	}
	function SubType() {}
	SubType.prototype = new SuperType();

	var instance1 = new SubType();

	instance1.colors.push("black");

	var instance2 = new SubType();
    console.log(instance2.colors); //"red,blue,green,black"
```

由于color属性是继承自SubType的原型，是引用类型值，当instance1修改其color属性时，实际上是修改的SubType原型里的color属性，由与instance2也继承了原型的color属性（其没有实例属性color）,所以instance2.color指向的color属性已经是被instance 修改过的了。

本来我们将属性放到构造函数里是为了其私有性考虑，可见这里的问题实质是属性的私有行被破坏了。


* 2. 创建子类实例时，不能向父类构造函数中传参。


解决以上两个问题，我们可以通过 "类抄写"（或叫"借用构造函数"）的方式来实现。 （具体叫什么不重要，名字只是一个代号，方便我们交流，我们只做的就是知道他是什么）。


### 借用构造函数（类抄写）

这种方法具体做法是：在子类构造函数内部通过call或apply方法调用父类构造函数。这样就可以在实现属性继承的时，既能传参，又能保证实例属性的私有行。
 
	
``` javascript
	function SuperType() {
		this.colors = ["red", "blue", "green"];
	}
	
	function SubType() {
	   SuperType.call(this);
	}
	
	var instance1 = new SubType();
	instance1.colors.push("black");
	alert(instance1.colors);    //"red,blue,green,black"
	
	var instance2 = new SubType();
	alert(instance2.colors);    //"red,blue,green"
```	

在SubType子类里通过SuperType.call(this) 调用父类，当实例化子类时，就将父配实例上的实例方法都赋值给了子类实例。

类抄写也有其问题，方法都在构造函数中定义，无法函数复用，浪费内存。
同时，在父类原型的属性对子类而言也是不可见的。所以实际应用中也很少单独使用借用构造函数的技术。而更常见的是将原型链和借用构造函数结合到一块，从而发挥两者之长的一种继承模式，及所谓的`组合继承`。这里就不展开细说了。
                  

### 使用Object.create()实现原型继承
ES5中新增Object.create() 来实现原型式继承。他使得原型继承不必依赖于构造函数，不用创建特定的对象类型。
这里我们只需要关心Object.create() 只有一个参数时即可（第二个参数和继承无关）。他返回一个新的对象，新的对象的原型对象即传入的Object.create()第一个参数。

	


### 原型继承的本质 
    
原型其实也是一个对象实例。 原型的含义：如果构造函数有一个原型对象A，则有构造函数创建的实例都必然复制自A（先不考虑这里具体的复制实现问题）。

"原型也是对象实例" 这是一个最关键的性质。这使得很自然的理解原型对象也有其自己的原型对象，即原型对象的原型对象。这是原型继承和类继承体系在本质上的不同。 对于类继承体系，类不必是"对象"，因此也不被对象的性质。 举例来说，“类”可以是一个内存块，也可能是一段描述文本，而不必是一个有对象特性（如可以调用方法或存取属性）的结构，（其实es6的class并不是通常意义上的class他还是对象）。

原型继承是一个典型的`以时间换空间`的解决方案。由于子类中直接读写一个成员而有无法存取到改成员时，将会回溯原型链以查找该成员的名字，因此直接结果是：继承层次中临近的成员访问更快，而试图访问一个不存在的成员时最耗时。

而现实的对象系统，我们其实更希望基类实现尽可能多的功能，希望通过较多的继承层次来使得类的粒度变小以便于控制。 从这里来看，访问更多的层次以及访问父类成员是复杂对象系统的基本特性。

而js 里更多层次的继往往承意味着属性的存取要经过更多层次的查找，更多的cpu消耗。这跟js 创立初期其所运行的环境有关，空间占用是关键，时间消耗次要的多。而在现在硬件条件下的js运行环境这些问题已经不再明显。


### 扩展-默认的原型 
这里借用一张网图：
![继承关系图](https://ws1.sinaimg.cn/large/d16dcf79ly1fpy2d5hfiqj20qy0z6tk1.jpg)

理解上图有几个关键事实：
> 原型对象也是一个实例对象
> <br/>构造函数也是一个实例对象

而实例对象都有自己的原型对象 ，通过__proto__属性访问（浏览器环境）。
构造函数的默认的原型对象是一个空的对象（empty objects），它是一个由Object创建的实例对象，所以其__ proto __ 指向Object.prototype。
Foo.prototype.__ proto __ === Objtct.prototype。

而构造函数又是谁创建的呢？所有的构造函数（包括内置对象，但除了Function它自身）都是由Function构造函数创建的，故构造函数也是个实例对象也有其原型，Foo.__ proto __ === Function.prototype。

同时Object也是Function 创建的，故Object.__ proto __ === Function.prototype。而Function.prototype 是一个Object创建的空的对象，故Function.prototype.__ proto __ === Object.prototype。

我们发现原型对象的源头是Object.prototype,那么Object.prototype 有自己的原型对象吗？有： Object.prototype.__ proto __ === null。








		




