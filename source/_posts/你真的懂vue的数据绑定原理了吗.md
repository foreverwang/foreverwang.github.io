---
title: 你真的懂vue的数据绑定原理了吗
date: 2019-07-05 16:21:50
categories: 
- javascript 
- vue
tags: 
- vue
- 数据绑定原理

---

## 你真的懂vue的数据绑定原理了吗？

### 前言
数据绑定有很多种实现方案，vue中的实现只是众多实现中的一种。都知道vue数据绑定是通过Object.defineProperty() 实现，但是在面试过程中，真正清楚vue数据绑定具体实现的不多，而网上各种vue原理解析的文章很多是二手三手的解读，千篇一律很多关键点都没有说到，本文主要是整理下自己的理解，并试图让不完全懂vue数据绑定原理实现的人看了能彻底懂。

### 数据绑定最终达到的效果
先了解数据绑定绑定的是什么？是UI。
数据绑定要达到的效果就是：UI渲染的时候使用数据对象(data)里的数据初始化UI，当UI中使用到的数据变化时，这个数据对应的UI会同步更新，而无需开发者手动修改DOM。
下面行文中，数据我们用一个命名为data的对象代指；而UI对应开发者直接接触的就是描述DOM部分的代码，	可能是一段html模板，也可能是一段jsx。

<!-- more -->

### 数据绑定的实现
#### 先上一个张原理图

![image](https://user-images.githubusercontent.com/25282685/60658801-8246e000-9e86-11e9-89aa-ebb5ede00830.png)


#### 知识储备（如果已经熟练掌握可跳过）：

- 访问器属性和Object.defineProperty() 

	defineProperty() 可以定义属性的getter和setter方法，当访问访问器属性的时候会调用该属性的getter方法，这样我们就可以知道全部的数据访问，并在此时做一些额外的工作。

		var data = { a: 1 }
		
		Object.defineProperty(data,'_a', {
		    get() {
		      console.log(`属性_a被访问了，在这里可以劫持这个数据访问的操作，返回我们想返回的值，并做些额外操作`);
		      return this.a;
		    },
		    set(newVal) {
		      console.log(`属性_a被赋值了，我们可以在此时劫持改操作`);
		      this.a = newVal
		    }
		})
		console.log(data._a)// 1, 属性_a被访问了...
		data._a = 2; //属性_a被赋值了...
		console.log(data._a); //2, 属性_a被访问了..

定义了getter setter的属性即成了 访问器属性。上例中属性 '_a'就是访问器属性。 当我们访问访问器属性的时候会先调用我们定义的getter方法，当赋值访问器属性的时候会先调用我们定义的setter方法。

这里可能有人会疑问，为什么不直接通过Object.defineProperty 定义属性'a',而是定义了一个和属性'a'关联的新属性'_a' ? 有这个疑问的同学建议在浏览器控制台执行以下上面代码（将被定义的属性改成'a'），可能就一下明白了： 这样当我们访问属性a的时候会造成死循环，因为我们将属性a定义为访问器属性，当访问属性a的时候会调用getter方法getter 方法内执行 `return this.a`时又访问了属性a,导致又调用了setter方法，然后死循环。

你可能还是有疑惑，如果我们想将一个对象的属性转成访问器属性那岂不是只能新增属性，那这对数据使用方十分不友好。答案是我们可以通过闭包变量实现不增加新属性而将原属性定义成访问器属性， 而我们还是访问原始的属性。具体做法看下边。
	
- 闭包变量

		var data = {a:3};
		function defineReactive(data, key, val) {
		    Object.defineProperty(obj, key, {
		        get() {
		          return val;
		        },
		        set(newVal) {
		          val = newVal;
		        }
		    })
		}
		defineReactive(data, 'a', obj['a'])
这里我们就将属性'a'定义成了访问器属性。这里方法名defineReactive直译过来叫「定义响应式」，也比较好理解：正是访问器属性的setter和getter可以劫持属性的读写操作，我们可以响应这些操作，故叫定义响应式，这样的数据也叫响应式数据。

示例中defineReactive 内部 变量val即是一个闭包变量，函数调用结束后不会被释放，因为在getter中有引用。所以当我们重新赋值属性a的时候 会将新值赋给闭包变量val,再次访问属性a的时候执行getter方法 返回闭包变量val的值。

- 发布/订阅模式

发布/订阅模式是一个再常见不过的模式了，这里不做赘述。下文提到的依赖收集及通知依赖即是发布订阅模式。

#### 原理解读	

现在回到上面的原理图。这里相对于源码我做了最大可能的精简，只保留了核心角色，这有是为了降低我们的理解负担。

首先看到这里有两个角色： 数据（data）, 和订阅者（watcher）。 
最开始我们对data做了一个操作：定义响应式即定义getter和setter。 这是vue整个数据绑定实现的最关键的一步。

我们的数据变成了响应式这意味着，数据的访问和修改我们都能知道，并在这个时机做一些操作。当访问某个数据（比如data对象的a属性）时，会触发这个数据的getter方法，我们在此将用到这个数据的订阅者（watcher）存起来；当这个数据发生变化时即修改其值时会触发该数据的setter方法，我们在此通知这个数据的订阅者们，订阅者们收到变化后会更新自己。

这里我们说一下订阅者（watcher）。

订阅者自身都有一个更新自己的方法（update）,当接收到通知后会调用自身update 方法，更新自己。

每个数据会有多个订阅者，UI中每当一处用到了该数据，就会添加一个该数据的订阅者。
举个小栗子：
我们的data对象如下：
	
	data = {a:1}

在模板中我们有一处用到了这个数据：

	<div> 第一次使用数据a:{{a}} </div>


当我们解析模板的时候访问到数据a, 就会触发数据a的getter,在其getter里我们将添加一个订阅者至数据a的订阅者列表。这里的订阅者就是这个div标签，前面我们说了每一个订阅者都有一个更新自己的方法。 当数据a发生变化是会触发其setter,在setter里我们通知数据a的订阅者们，这些订阅者们收到通知的时候再调用自己的更新方法。

注意：在vue 1.x 里面当触发数据更新的时候是直接通知到其订阅者使其订阅者更新，这个更新粒度是节点级的。而vue2.x里加入了虚拟DOM，这个流程发生一点变化：状态侦测不再细化到某个具体节点，而是某个组件组件内部通过通过虚拟DOM来渲染视图。不过数据绑定的原理并没有变化。关于虚拟DOM，我会另开[一篇]()。


### 关于源码
这里并不会逐行解读源码，这类的文章网上一大片，而且篇幅会很长，我们这里只是结合源码的实现对上文做些补充，以及对源码里的我认为的巧妙之处单独说下。

通过上面或许已经大致能明白vue里数据绑定的实现了。而此时你去看源码的时候，会发现在有一些差异。在源码中数据绑定相关的还多了Observer类、Dep类。

Observer类。其实我觉得这个命名不好，它所做的事情就是给所有数据添加getter和setter将其改写成响应式数据，所以我这里直接叫「定义响应式」语义上会更准确，对应我们上图中defineReactive。Observer在通俗意义上更多的是指观察者模式里的观察者，而发布订阅模式即是观察者模式的一个变体，所以还容易造成理解上困扰。（比如我第一次看网上的原理图看到Observer时，还以为数据添加订阅者是在这个类里操作的)。

Dep类。它是和Watcher紧密关联的。 它是作用是用来为每个数据存放其订阅者，当这个数据变化了再通知这些订阅者们更新。可以看到Dep就是一个中介者。以租房中介为例：某个数据是一个房东，Dep是中介，租房者们是一个个watcher。当有人租房的时候都找Dep,当房东要收钱的时候告诉中介，中介负责通知各个租客交房租。

这里又一次用到了闭包变量：

		function defineReactive(data, key, val) {
			let dep = new Dep(); //闭包变量
		    Object.defineProperty(obj, key, {
		        get() {
		        		dep.depend(); //添加订阅者
		          	return val;
		        },
		        set(newVal) {
		          	val = newVal;
		          	dep.notify(); //通知订阅者更新
		        }
		    })
		}
		
		let data = {a: 1};
		defineReactive(data, 'a', obj['a']);

这里我们调用defineReactive方法将data的属性a定义响应式，这里产生一个闭包变量dep, 而其是和属性a绑定的，后续读写属性a会调用属性a的getter/setter，会用到dep 变量。每访问一次属性a都对会对a添加一个订阅者。


看defineReactive方法，你可能会有疑问的：dep.depend() 具体添加的是什么？答案是Watcher实例。但此时你可能还有疑问：在定义dep.depend 方法的时候我们还不知道watcher长什么样呢，因为watcher是有多种类型的（比如更新文本值、更新指令值等），当我们在需要添加的那一刻才知道订阅者长什么样。那怎么做到 dep 可以不事先关心 watcher的具体实现，在需要添加的地方又能自动将watcher添加到dep里呢？ 

直线的思维是当我需要添加watcher的时候，调用一下dep.depend(watcher)。单这样有一个问题： 当我们在外部调用dep.depend(watcher)，就要知道每一个数据其对应的dep如何访问到，这可以定义一个全局对象来存放每个数据的dep，但这样其实并不优雅，封装性不够好。所以我们看到vue 源码里用一个闭包变量来存放每个数据对应的dep，而闭包变量在外部我们访问不到。 接下来我们就看一下vue 里是怎么做的，十分之巧妙。

下面为便于理解用的是伪代码，并不是vue 里源代码。

Dep的depend方法

	depend() {
		if (Dep.target) {
			this.addSub(Dep.target);
		}
	}
depend方法判断了一个条件 Dep.target如果存在则添加将Dep.target添加到订阅者队列里。


Watcher里get方法（会在constructor 调用）
   //为数据添加订阅者
	get() {
		Dep.target = this;
		let value = this.getter(vm，expOrFn,cb); // 会触发数据的getter 方法， 可以先不关心它具体做了啥
		Dep.target = null;
		return value;
	}
	
当解析到UI里用到了某一个数据后，我们就new Watcher(vm，expOrFn,cb)得到一个watcher实例, Watcher的constructor 会执行其get方法，该方法里先将Dep.target赋值为当前watcher实例，接着去访问这个数据，就会触发这个数据的getter方法，在数据的getter 里我们调用dep.deepend方法，deepend方法内将Dep.target（此时就是watcher实例）添加到当前数据的订阅者队列。 然后再将Dep.target置空。

我们再回顾下前面的定义响应式的方法对每个数据定义的getter：


	function defineReactive(data, key, val) {
		let dep = new Dep(); //闭包变量
	    Object.defineProperty(obj, key, {
	        get() {
	        		dep.depend(); //添加订阅者
	          	return val;
	        },
	        set(newVal) {
	          	//....
	        }
	    })
	}

这就保证了只有通过new Watcher去访问的数据才会被添加到该数据的订阅者队列。
比如我们在代码里写 `data.a` 这样去访问属性a时，由于Dep.target是null,所以不会被添加无意义的订阅者。

至此数据绑定原理基本上讲完了。 下面附上数据绑定相关部分精简代码实现的完整版：

// Observer

	/**
	 * Observer会将数据对象的所有属性都转换为getter/setter的形式来收集依赖（前面我们说的订阅者），并当属性值变化时通知这些依赖
	 */
	
	export default class Observer() {
		constructor(value) {
			this.value = value;
			if (!Array.isArray(value)) {
				this.walk(value);
			}
			// 注意数组的处理方式略有不同，这里不暂且不包括数组的处理
		}
	
		walk(obj) {
			const keys = Objects.keys(obj);
			for(let i=0, l = keys.length; i<l; i++) {
				defineReactive(obj, keys[i], obj[keys[i]])
			}
		}
	}
	
	
	// defineReactive
	function defineReactive(data, key, val) {
		// 递归子属性
		if (typeof val === 'object') {
			new Observer(val);
		}
		let dep = new Dep();
		Object.defineProperty(data, key, {
			enumerable: true,
			configulable: true,
			get() {
				dep.depend();
				return val;
			},
			set(newVal) {
				if (val === newVal) {
					return;
				}
				val = newVal;
				dep.notify();
			}
		})
	}


// Dep

		export default class Dep {
			constructor() {
				this.subs = [];
			}
			addSub(sub) {
				this.subs.push(sub);
			}
			removeSub(sub) {
				remove(this,subs, sub);
			}
			depend() {
				if (Dep.target) {
					this.addSub(Dep.target);
				}
			}
			notify() {
				const subs = this.subs.slices();
				for (let i=0,l=subs.length; i<l; i++) {
					subs[i].updates();
				}
			}
		}
		
		function remove(arr, item) {
			if (arr && arr.length) {
				const index = arr.indexOf(item);
				if (index > -1) {
					return arr.splice(index, 1)
				}
			}
		}

// Watcher

		export default class Watcher {
			constructor(vm, expOrFn, cb) {
				this.vm = vm;
				this.getter = parsePath(expOrFn);
				this.cb = cb;
				this.value = this.get();
			}
			get() {
				Dep.target = this;
				let value = this.getter.call(this.vm, this.vm);
				Dep.target = null;
				return value;
			}
			update() {
				const oldValue= this.value;
				this.value = this.get();
				this.cb.call(this.vm, this.value, oldValue);
			}
		}
		
		const bailRe = /[^λw.$]/
		function parsePath(path) {
			if (bailRe.test(path)) {
				return;
			}
			const segments = path.split('.');
			return function (obj) {
				for (let i=0, l=segments.length; i < l; i++) {
					if (!obj) return;
					obj = obj[segments[i]];
				}
				return obj;
			}
		}
	

#### 参考资料

- vue2.6[源码](https://github.com/vuejs/vue/tree/2.6)
- 最后精简版源码部分来自《深入浅出Vue.js》

