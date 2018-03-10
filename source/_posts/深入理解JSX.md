---
title: 深入理解JSX
date: 2017-07-10 00:06:54
categories: 
- javascript 
- React
tags: 
- JSX 
- React
---

### JSX 是啥?

>  JSX = JS + XML 

JSX提供了一种可以在JS里写XML的语法。

### JSX的运行环境 
> 没有环境可以直接直接执行JSX代码 

<!-- more -->

最终JSX代码在运行前都被转换成了JS。转换方式：
 
  * 静态编译器编译（工具babel）
  * 运行期通过jsxtransform.js 进行转换。（已被facebook废弃，性能问题）

思考：浏览器或者js引擎为什么不直接支持jsx? -->继续看下边

### JSX的由来

JSX是伴随着facebook的javascript类库 react 的出现被发明的。jsx 的出现和虚拟DOM有直接关系的。
##### 一句话介绍虚拟DOM：
>在内存中创建的描述DOM节点的js对象。为了减少对实际DOM的操作从而提升性能。（这里可以先不管什么是虚拟DOM）

##### 如何创建虚拟DOM：

``` javascript 
    var child1 = React.createElement('li', null, 'First Text Content');
    var child2 = React.createElement('li', null, 'Second Text Content');
    var root = React.createElement('ul', { className: 'my-list' }, child1, child2); 
```

==>   '虚拟Dom对象 ' ==>  

``` html
    <ul class="my-list">
      <li> 'First Text Content' </li>
      <li> 'Second Text Content' </li>
    </ul>
```

使用这样的机制，我们完全可以用JavaScript构建完整的界面DOM树，但是代码编写麻烦，可读性差（`IOS类似，ios是否可以借鉴jsx?`）。于是React发明了JSX，利用更友好的HTML语法来创建虚拟DOM：


``` html
      var root =(
          <ul className="my-list">
            <li>First Text Content</li>
            <li>Second Text Content</li>
          </ul>
    );
```
##### 一句话总结jsx存在的意义
> 让我们更直观的愉快的写代码。

前端界面的最基本功能在于展现数据，为此大多数框架都使用了模板引擎，就对应了自己的模板语法：如angular.js


``` html
    <div ng-if="person != null">
        Welcome back, <b>{{person.firstName}} {{person.lastName}}</b>!
    </div>
    <div ng-if="person == null">
        Please log in.
    </div>
```
jsx的优势之一就是不需要掌握一门模板语法。如果说掌握一种模板语言并不是很大的问题，那么其实由模板带来的架构复杂性则是让框架也变得复杂的重要原因：
http://www.infoq.com/cn/articles/react-jsx-and-component

React直接放弃了模板而发明了JSX。

### JSX的语法  
这里不会对jsx语法详细介绍，只介绍一些注意点。[详情](https://facebook.github.io/react/docs/jsx-in-depth.html)

#### 一、通过{}插入js表达式 

``` html
    var person = <Person name={window.isLoggedIn ? window.name : ''} />;
```

    这个功能就很强大了，你可以尽情的使用js的功能： 

#### 二、组件tag 大小写敏感
这里还不是一般的敏感，大小写是有限制的。所有对应证实节点的tag 要小写开头，所有组件节点tag 要大写开头。大小写是告诉转换工具（babel）要如何转换。我们看个例子：


``` html 
    React.render(<div>xxx</div>)
    
    React.render(<App/>)
```

 转换后==>


``` javascript 
    React.render(React.createElement(
      "div", //小写tag会编成字符串
      null,
      "xxx"
    ));

    React.render(React.createElement(App, null));//大写编成变量
```
一句话总结标签大小写注意点：  
> 可以不用关心这些细节，这些都是给框架用的，只需要知道何时大写何时小写就好。
    
#### 三、绑定事件
##### 绑定方式：

``` html
     <div onClick={this.handler.bind(this)}>Submit</div>
```

##### 无需手动解绑事件
在JSX中你不需要关心什么时机去移除事件绑定，因为React会在对应的真实DOM节点移除时就自动解除了事件绑定。

##### 事件代理
React并不会真正的绑定事件到每一个具体的元素上，而是采用事件代理的模式：在根节点document上为每种事件添加唯一的Listener，然后通过事件的target找到真实的触发元素。这样从触发元素到顶层节点之间的所有节点如果有绑定这个事件，React都会触发对应的事件处理函数。这就是所谓的React模拟事件系统。

#### 四、在JSX中使用样式
通常情况下我们应该把样式写到css文件里，当有时对于特定组件而言其样式比较简单且或固定，那么可将其直接写在jsx中。在jsx中通过style属性来定义，但和真实DOM不同的是，属性值不能是字符串而必须为对象。

``` html
    <div style={{color: '#ff0000', fontSize: '14px'}}>Hello World.</div> 
```
这里有两层大括号，外面的大括号是JSX的语法，变的大括号是js对象。

##### 属性名转驼峰
在JSX中可以使用所有的的样式，基本上属性名的转换规范就是将其写成驼峰写法，如background-color --> backgroundColor。
 
 
### JSX 与babel 
如何让babel将JSX传换成js时指定自定义方法名而不是React.createElement()。
这个babel支持配置：.babelrc文件 [详情](https://babeljs.io/docs/plugins/transform-react-jsx/)

``` json
    {
          "plugins": [
            ["transform-react-jsx", {
              "pragma": "dom" // default pragma is React.createElement
            }]
          ]
    }
```


