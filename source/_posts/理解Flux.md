---
title: 理解Flux
date: 2017-09-16 21:16:08
tags:
 - flux
 - 数据流
categories: 
 - 前端状态管理
---

### 什么是flux
flux是一种架构模式，用来指导（或约束）我们的软件结构。其核心是“单向数据流模式”。同MVC架构是同样的存在。
(举例 ：React 说自己是 MVC 里面 V 的部分，那么 Flux 就相当于添加 M 和 C 的部分)

### 单向数据流
Flux 的核心“单向数据流“的结构和流程：


>   Action --> Dispatcher --> Store --> View

<!-- more -->

* 首先要有 action，通过定义一些 action creator方法 根据需要创建 Action。
* View 层通过用户交互（比如 onClick）会触发 Action
* Dispatcher 会分发 所触发的 Action 到所有注册的 Store 的回调函数。
* Store 回调函数根据接收的 Action 更新自身数据之后会触发一个 change 事件通知 View 数据更改了
* View 会监听这个 change 事件，拿到对应的新数据并 更新组件 UI(如调用 setState)


### 基本概念

* View： 视图层 （视图组件）
* Action（动作）：视图层发出的消息 ：包括消息类型和消息的数据。
* Dispatcher（派发器）：用来接收Actions、执行回调函数。
* Store：包含应用状态和逻辑。

#### Action （每个Store有一堆）

每个Action都是一个对象，包含一个actionType属性（说明动作的类型）和一些其他属性（用来传递数据）。



#### Dispatcher (全局一个)
Dispatcher 是中心枢纽，其`作用`是将 Action 派发到 Store，管理所有数据流向。Dispatcher `本质`上就是一个事件系统。
你可以把它看作一个路由器，负责在 View 和 Store 之间，建立 Action 的正确传递路线。
以Facebook 的 Dispatcher 库为例，Dispatcher有两个核心方法：

*  dispatch：派发action到store 注册的回调。
*  register： store 会调用该方法注册其回调，回调里对不同的action做处理。

Dispatcher 只用来派发 Action，不应该有其他逻辑。且Dispatcher 只能有一个，而且是全局的。
各Store在dispatcher上注册自己的回调，这样dispatcher上就有一张回调注册表，与各Store建立联系。


#### Store（多个） 响应action(事件)
Stores 包含应用的状态和逻辑，不同的 Store 管理应用中不同部分的状态。
Store响应 Dispatched Actions （被分发的事件）。
重点：应用中唯一知道如何更新数据的就是 Store。
store在dispatcher上注册的回调接受一个action参数，回调里面是一个switch语句，根据action的type分发给具体state更新方法，store更新完毕后，通过广播事件来告诉view某些状态变了，对应的view取新的状态更新自己。


#### view （多个）

接收用户操作触发action。 action通过dispatcher派发到store里处理更新数据状态，并发出改变的通知，view获取新的状态更新自己。



### 有什么好处？

这里引用尤雨溪知乎的一个回答：

> 1. 视图组件变得很薄，只包含了渲染逻辑和触发 action 这两个职责，即所谓 "dumb components"。（`个人理解：数据层和view层分离`）
> 2. 要理解一个 store 可能发生的状态变化，只需要看它所注册的 actions 回调就可以。（`个人理解：和1说的一回事，关注点分离后，是的结构更清晰，代码可读性和可维护性提高`）
> 3. 任何状态的变化都必须通过 action 触发，而 action 又必须通过 dispatcher 走，所以整个应用的每一次状态变化都会从同一个地方流过。其实 Flux 和传统 MVC 最不一样的就在这里了。React 在宣传的时候一直强调的一点就是 “理解你的应用的状态变化是很困难的 (managing state changing over time is hard)”，Flux 的意义就在于强制让所有的状态变化都必须留下一笔记录，这样就可以利用这个来做各种 debug 工具、历史回滚等等。（`个人理解：使数据可预测`）



### 参考资料
具体实例可以参考阮老师的这边文章 ：[Flux 架构入门教程](http://www.ruanyifeng.com/blog/2016/01/flux.html)

这篇总结的非常到位，但是前提是得你先大概理解了flux：[flux总结](http://www.ayqy.net/blog/flux/)

facebook 官方提供的 [dispatcher](https://github.com/facebook/flux/blob/master/src/Dispatcher.js)





