---
title: 前端MVC
date: 2017-10-05 15:49:16
tags: 
 - MVC
 - MVP
categories:
  - 架构模式
  - 设计模式

---

### 前言

本文是个人对前端MVC的理解。当我去查MVC资料的时候发现，一千篇文章有一千种MVC，搞的我很纠结。于是研究总结为本文。如果要先一句话介绍什么是MVC，那么：

> MVC是开发GUI（图形用户界面）程序的一种架构模式。

<!-- more -->

是的，我这里想要搞清的就是`前端MVC`（包括web前端，客户端开发），不涉及服务端MVC。  

### 说说历史
MVC模式最初是在1979年由 TrygveReenskaug在研究Smalltalk-80期间设计出来的。本文称之为经典MVC（以和现代的一千种MVC相区别）。

这么多年过去了，如今的软件环境和当年已大不相同，所以我们今天所看到的MVC已不同往日。比如经典MVC中说
> “view永远不应该知道用户输入，比如鼠标操作和按键。”  --经典MVC

显然在今天的前端开发中是做不到的，用户输入必须通过监听view上的事件来获得。

因为当年View的功能及其弱，比如鼠标的光标都需要程序自己绘制。经典MVC中，Controller要做的事情多数是派发用户输入给不同的View，并且在必要的时候从View中获取Editor来更改Model，而Web以及绝大多数现在的UI系统中，Controller的这部分职责已经被系统实现了。即现在的view 系统实现的更强大了，比如view自身能响应事件并带有一些事件数据。

> 经典MVC中称"controller是用户和系统之间的链接"，也就不难理解解了。

所以，如今 view.onclick = ... ，在当年就是 mouse.onclick = ...  


知道了当年和现在软件环境的差异后，那么我们就重点关注那些些没变的东西就好了，也没要去纠结 Smalltalk-80 MVC最初的实现了。变的是环境，不变的是思想。

> MVC强制将业务数据（Model）与用户界面（View）隔离，Controller管理业务逻辑和用户输入。


### MVC
接下来介绍 MVC的各个部件：

#### Model 
Model 代表特定领域数据？？，不了解用户界面（view 和controller)；当一个model 发生变化时，他会通知他的观察者（view）；

#### View
View 描绘的是Model的当前状态。view 通过观察者模式观察 model以了解model何时更新。

#### Controller
在view-controller 对中作用是处理用户交互（如按键和点击等动作），为view 做决定。



#### MVC的依赖关系

![mvc依赖关系图](https://ws1.sinaimg.cn/large/006tKfTcgy1fk7cgu66arj30kk0cwq40.jpg)

Controller和View 都依赖Model: view 的数据来源为model，controller 调用model的 数据处理方法。 controller和view互相依赖,在一些网上的资料Controller和View之间的依赖关系可能不一样，有些是单向依赖，有些是双向依赖，这个其实关系不大，后面会看到它们的依赖关系都是为了把处理用户行为触发的事件处理权交给Controller。

#### MVC调用关系

![MVC调用关系](https://ws4.sinaimg.cn/large/006tKfTcgy1fk7cx1rbn7j30j80kkgni.jpg)

* 用户交互来源主要是view (或者输入url),view 捕获到操作后，将处理权委托给controller,controller对来自view 的数据预处理，决定调用那个model的那个接口（应用逻辑）；然后有model 执行相应的业务逻辑； model 更新后，通过观察者模式通知view ;view 接收到通知后向model请求最新数据（或者model主动传递）更新自己。

#### MVC实例 
    http://jsfiddle.net/uVBvq/


``` html
    <form action="">
        <span id="num"></span><br />
        <button type="button" id="increase">+</button>
        <button type="button" id="decrease">-</button>
    </form>
    
    <script>
    var myapp = {};

    myapp.Model = function () {
        var val = 0;
        this.add = function (v) {
            if (val < 100) val += v;
        };
        this.sub = function (v) {
            if (val > 0) val -= v;
        };
        this.getVal = function () {
            return val;
        };
        
        //model拥有 view 的引用？
        var views = [];
        this.register = function (view) {
            views.push(view);
        }
        
        //model 变化时通知model更新
        this.notify = function () {
            for (var i = 0; i < views.length; i++) {
                views[i].render(this);
            }
        };
    };
    
    //view 拥有controller的引用
    myapp.View = function (controller) {
        var $incBtn = $('#increase');
        var $decBtn = $('#decrease');
        var $num = $('#num');
    
        this.render = function (model) {
            $num.text(model.getVal() + 'px');
        };
    
        $incBtn.click(controller.increase);
        $decBtn.click(controller.decrease);
    };
    
     
    myapp.Controller = function () {
        var model = null;
        var view  = null;
    
        this.init = function () {
            //controller初始化 model 和view
            model = new myapp.Model();
            view  = new myapp.View(this);
           
           //view 向model 注册自己
            model.register(view);
            model.notify();
        };
        
        //controller 负责更新model 
        this.increase = function () {
            model.add(1);
            model.notify();
        };
        this.decrease = function () {
            model.sub(1);
            model.notify();
        };
    };
    
    
    //外界只接触到controller 
    $(function () {
        var controller = new myapp.Controller();
        controller.init();
    });
    </script>
```
### MVC优缺点 
#### 优点
* 耦合性降低：视图和模型分离，利于项目工程化（分工等）。
* Model可复用性增强： 因为Model是独立于view的。

#### 缺点  （[以下直接引用](https://github.com/livoras/blog/issues/11)）
* Controller测试困难。因为视图同步操作是由View自己执行，而View只能在有UI的环境下运行。在没有UI环境下对Controller进行单元测试的时候，应用逻辑正确性是无法验证的：Model更新的时候，无法对View的更新操作进行断言。
* View无法组件化。View是强依赖特定的Model的，如果需要把这个View抽出来作为一个另外一个应用程序可复用的组件就困难了。因为不同程序的的Domain Model是不一样的。 

### MVC其他
#### ios里的MVC（todo）

kvo ...

#### 服务端MVC 
服务端也有MVC，但是经典的MVC模式只是解决客户端图形界面应用程序的问题，而对服务端无效，服务端MVC的变种也有自己的一个名字 MVC Model2 这里不介绍了。

### MVC衍生--MVP
顺便把mvp说一下吧。MVP是MVC的衍生品。MVP有两种：
 
 * Passive View
 * Supervising Controller
 
常见的是Passive View（被动视图），这里也只说这种。
 
#### MVP依赖关系

![MVP依赖关系](https://ws2.sinaimg.cn/large/006tKfTcgy1fk7e9b9vhjj30ii0b60tn.jpg)

MVP打破了View原来对于Model的依赖，其余的依赖关系和MVC模式一致。

#### MVP（Passive View）的调用关系

![MVP（Passive View）的调用关系](https://ws4.sinaimg.cn/large/006tKfTcgy1fk7edqp1frj310u0t8jts.jpg)

view和model 完全解耦后，model更新是如何同步view的呢？ 是通过presenter。
mvc 中是model更新后通知view更新。mvp中是model更新后，通过观察者告知presenter而不是view了。presenter获取到Model变更的消息以后，通过View提供的接口更新界面。

#### MVP（Passive View）的优缺点（[以下直接引用](https://github.com/livoras/blog/issues/11)）


##### 优点：

* 便于测试。Presenter对View是通过接口进行，在对Presenter进行不依赖UI环境的单元测试的时候。可以通过Mock一个View对象，这个对象只需要实现了View的接口即可。然后依赖注入到Presenter中，单元测试的时候就可以完整的测试Presenter应用逻辑的正确性。这里根据上面的例子给出了Presenter的单元测试样例。
* View可以进行组件化。在MVP当中，View不依赖Model。这样就可以让View从特定的业务场景中脱离出来，可以说View可以做到对业务完全无知。它只需要提供一系列接口提供给上层操作。这样就可以做到高度可复用的View组件。

##### 缺点：

* Presenter中除了应用逻辑以外，还有大量的View->Model，Model->View的手动同步逻辑，造成Presenter比较笨重，维护起来会比较困难。

//todo: 待实例验证优缺点 
 
    
### 资料
* winter http://www.cnblogs.com/winter-cn/p/4285171.html
* EFE http://efe.baidu.com/blog/mvc-deformation/
* https://github.com/livoras/blog/issues/11
* https://speakerdeck.com/jaceju/understanding-the-mvc-mvp-mvvm-in-javascript

    
