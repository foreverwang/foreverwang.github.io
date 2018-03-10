---
title: 理解redux
date: 2017-12-06 15:38:51
tags:
    - redux
    - 数据流
categories:
    - 前端状态管理
---

### redux简单介绍
前面[理解了flux](),本篇来理解redux。
redux的官方介绍 
> Redux is a predictable state container for JavaScript apps.

翻译一下：redux 是javascript 应用状态容器，他提供可预测化的状态管理。
师出flux,作用和flux一样，作为应用的状态管理层。其核心思想也是单向数据流。

我们先看一个redux 最简单的原生使用实例；然后再胡乱解读一通redux概念；最后通过源码来看下其实现原理。

<!-- more -->

### redux如何使用-代码示例 
[Counter Vanilla](https://github.com/reactjs/redux/blob/master/examples/counter-vanilla/index.html) 这是redux 官方一个原生代码使用redux的实例，非常好懂。 例子是一个计数器，点击+号 完成加一操作，点击 -号，完成减一操作。


``` html
    <!DOCTYPE html>
    <html>
      <head>
        <title>Redux basic example</title>
        <script src="https://cdn.bootcss.com/redux/3.7.2/redux.js"></script>
      </head>
      <body>
        <div>
          <p>
            Clicked: <span id="value">0</span> times
            <button id="increment">+</button>
            <button id="decrement">-</button>
            <button id="incrementIfOdd">Increment if odd</button>
            <button id="incrementAsync">Increment async</button>
          </p>
        </div>
        <script>
          //counter函数是一个reducer，现在只需要知道reducer就是操作应用数据的方法
          function counter(state, action) {
            if (typeof state === 'undefined') {
              return 0
            }
            switch (action.type) {
              case 'INCREMENT':
                return state + 1
              case 'DECREMENT':
                return state - 1
              default:
                return state
            }
          }
          //实例化一个 store ，store里可以持久化存储应用的数据 state
          //并且对外暴露几个方法
          //其中 getState()用来获取应用的数据
          //其中 subscribe()用来添加订阅者，供数据发生变化时通知
          //其中 dispatch(action) 触发一个动作用来改变数据 
          var store = Redux.createStore(counter)
          var valueEl = document.getElementById('value')
          
          
          //render函数 将数据（state）呈现到view
          function render() {
            valueEl.innerHTML = store.getState().toString()
          }
          render()
          
          //将render 订阅store，数据变了的时候view重新渲染
          store.subscribe(render)
          
          //通过dispatch触发一个操作 来修改数据
          document.getElementById('increment')
            .addEventListener('click', function () {
              store.dispatch({ type: 'INCREMENT' })
            })
          document.getElementById('decrement')
            .addEventListener('click', function () {
              store.dispatch({ type: 'DECREMENT' })
            })
          document.getElementById('incrementIfOdd')
            .addEventListener('click', function () {
              if (store.getState() % 2 !== 0) {
                store.dispatch({ type: 'INCREMENT' })
              }
            })
          document.getElementById('incrementAsync')
            .addEventListener('click', function () {
              setTimeout(function () {
                store.dispatch({ type: 'INCREMENT' })
              }, 1000)
            })
        </script>
      </body>
    </html>

```

这个例子很简单。 

* 数据存储在store；
* view（render函数）订阅store,store变化的时候会通知view 进行刷新；
* 通过store.dispatch 来修改数据

我们的应用无非就是这么几部分：

* 用户界面（呈现数据）；
* 应用数据；
* 操作数据和响应用户操作的方法。 

这些即对应MVC架构的view、model、controller。
在flux、redux里，也有这些角色，那他们和MVC架构有何不同呢？那就是 flux、redux强制单向数据流来使得数据可预测。 下面我们来看redux的概念和结构。

### redux 结构

上面这个例子中，状态变化的数据流向是下边这样的：
>
           dispatch
     actions --> store | reducers  --> new state --> view 
      |                                               |
      <-----------------------------------------------

首先创通过createStore(counter)建了一个store。
点击加号的过程：触发一个加一的action: store.dispatch({ type: 'INCREMENT' }),dispatch 会把这个action分发到reducer（这里是counter)里。reducer 接收当前的应用状态state,返回更改后的state。 到这里数据就完成了一次修改。 由于我们的view 事先订阅了store（store.subscribe(render)）,此时store 会通知view state 变了，view 刷新（调用render方法）。
     
#### action 
##### action概念

和 flux里的概念一样，用来描述发生了什么。 
是一个普通对象，包含一个type属性来描述这个动作， 和其他数据属性。
比如 { type: 'INCREMENT', num: 1},描述一个增加的动作，增加值为1。
    
##### action 扩展

* createAction

    也和flux 里一样，是一个生成action的函数，输入action中变化的部分作为参数，输出一个action对象.因为同样的action 在应用中可能多次用到，通过createAction函数可以减少重复代码,提高代码复用。
    
* bindActionCreators 

    redux里还提供了bindActionCreators。action只是描述一个行为(好比一个事件event),他本身是不会产生任何影响的，想要触发一个action 让这个描述变成现实，是通过dispatch 方法来触发的（dispatch一个action 好比触发一个event）。而bindActionCreators 就是对dispatch 行为的一个封装，是的业务方触发一个action时可以像普通函数一样调用，而不感知dispatch 的存在。更具体的下面解读源码再说。


#### reducer 
reducer 是纯函数。
上面我们说了让一个action产生影响是通过dispatch(action)来触发的。 而这里的dispatch 只是触发而已，具体干活的就是reducer，dispatch内部调度reducer。
reducer是一个函数，接收当前state和action 返回新的state。

reducer 和 dispatch 共同充当了flux里的dispatcher（registerCallback,dispatch）角色。

##### reducer扩展
* combineReducers 方法
实际应用中有不止一个reducer。redux里提供了一个combineReducers()方法来组织这些reducer。具体下边源码分析再说。


#### store 
     
上面我们知道了，我们使用action 来描述发生了什么，使用reducer来根据action更新state。

而store 就是把action 和reducer结合起来的对象，并持有state。store负责：

* 维持应用的state,并提供getState()方法获取state
* 提供 dispatch(action) 方法更新 state；
* 提供 subscribe(listener) / unsubscribe(listener) 来注册和解绑监听者。 

Redux 应用只有一个单一的 store，当需要拆分数据处理逻辑时，你应该使用 reducer 组合 而不是创建多个 store。

### redux 源码解读(基于redux3.7)

#### 先看下代码结构 
redux 对外暴露五个方法，分别对应五个同名js文件：
``` javascript
      createStore.js //保存应用状态 定义内部方法，输出对外暴露的方法
      combineReducers.js //将多个reducer 合并成一个
      bindActionCreators.js //把actionCreator 包了一层dispatch，使用时可以像普通函数一样调用actionCreator
      applyMiddleware.js //组合串联middleware
      compose.js //工具方法，将middleware中间件方法组合成一个调用链
```
#### createStore.js
这是redux 最核心概念的实现部分。createStore方法的作用是创建一个store,并维持state。下面我们就通过源码看下store 是如何维持state的。


createStore方法接受三个参数：reducer [, preloadedState, enhancer],
返回一个store对象，包含4个方法：
    
* dispatch //触发action
* subscribe// 添加监听者
* getState //获取当前state
* replaceReducer 
* observable

以下源码为方便阅读，去掉了原注释，和异常抛错信息。注释信息为解读信息。

``` javascript

    import $$observable from 'symbol-observable'
    import ActionTypes from './utils/actionTypes'
    import isPlainObject from './utils/isPlainObject'
    /**
     * [createStore description]    通过createStore方法创建一个store,store来维护和持有应用的state
     * @param  {Function} reducer   函数 接收当前state和action为参数返回新的state
     * @param  {any} preloadedState 默认的初始化state，如果你的reducer是通过combineReducers生成的一个顶层reducer,
     *                              那初始state的key值和combineReducers的key是一一对应的
     * @param  {Function} enhancer  store增强器 通常添加中间件
     * @return {Store}              store，允许你读取state,dispatch actions以及注册订阅者
     */
    export default function createStore(reducer, preloadedState, enhancer) {
      //允许使用方灵活传参：可以不传默认state 直接传 enhancer参数
      if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
        enhancer = preloadedState
        preloadedState = undefined
      }
    
      if (typeof enhancer !== 'undefined') {
        if (typeof enhancer !== 'function') {
          throw new Error('Expected the enhancer to be a function.')
        }
        //这行代码到 applyMiddleWare时说明
        return enhancer(createStore)(reducer, preloadedState)
      }
    
      if (typeof reducer !== 'function') {
        throw new Error('Expected the reducer to be a function.')
      }
      
      //这里是一些函数内的变量，由于闭包特性，这些变量在调用createStore函数后不会被销毁（因为导出的对象方法里面有这些变量的引用）。
      //这里指的注意的一点是有两个listeners队列：currentListeners 和 nextListeners；这一点有点厉害了，下面介绍
      let currentReducer = reducer 
      let currentState = preloadedState
      let currentListeners = []
      let nextListeners = currentListeners
      let isDispatching = false
      
      //nextListeners是currentListeners的拷贝，我们修改（subscribe）都是对nextListeners的修改
      //在flush listeners 之前 nextListeners 同步 currentListeners
      //这么做可以保证 每次dispatch listener 过程中 subscribe/unsubscribe listerens不会影响当前dispatch 队列，改变值发生在nextListeners，下一次dispatch时生效。 
      //TODO：我只知道这里这么做有点屌，具体这么做有啥用，我还得研究下
      
      function ensureCanMutateNextListeners() {
      //nextListeners === currentListener 说明listeners已经被flush; 
      //nextListeners !== currentListener 说明subscribe(listener)后 还未被dispatch
        if (nextListeners === currentListeners) {
          nextListeners = currentListeners.slice()
        }
      }
        /**
         * 读取state 
         * @returns {any} 返回应用的当前state树
         */
      function getState() {
        if (isDispatching) {
          throw new Error('')
        }
    
        return currentState
      }
    
      function subscribe(listener) {
        if (typeof listener !== 'function') {
          throw new Error('Expected listener to be a function.')
        }
    
        if (isDispatching) {
          throw new Error('')
        }
    
        let isSubscribed = true
    
        ensureCanMutateNextListeners()
        nextListeners.push(listener)
    
        return function unsubscribe() {
          if (!isSubscribed) {
            return
          }
    
          if (isDispatching) {
            throw new Error('')
          }
    
          isSubscribed = false
    
          ensureCanMutateNextListeners()
          const index = nextListeners.indexOf(listener)
          nextListeners.splice(index, 1)
        }
      }
    
      //唯一改变store里数据（state）的方式就是调用store.dispatch
      //dispatch 做了两件事情：1调用 reducer 更改数据 2 flush 所有listeners
      //这里和flux有点不同, flux里 观察数据的变化，以及数据变化后的通知，这些需要你自己去实现 这一环节的观察者模式。
      //redux自身实现了这一环节的观察者模式，提供了 store.subscribe方法添加listeners，dispatch的时候会 flush这些listeners 
     
      function dispatch(action) {
        if (!isPlainObject(action)) {
          throw new Error('')
        }
    
        if (typeof action.type === 'undefined') {
          throw new Error('')
        }
        
         // reducer内部不允许再次调用dispatch，否则抛出异常 防止死循环？
        if (isDispatching) {
          throw new Error('Reducers may not dispatch actions.')
        }
    
        try {
          isDispatching = true
          currentState = currentReducer(currentState, action)
        } finally {
          isDispatching = false
        }
        
        //触发所有的状态监听回调函数
        const listeners = currentListeners = nextListeners
        for (let i = 0; i < listeners.length; i++) {
          const listener = listeners[i]
          listener()
        }
    
        return action
      }
      
      // 顾名思义，就是替换当前store在用的reducer
      // 有什么用呢？你想动态加载某些reducer时,加载后用该方法替换reducer; 如果你想为redux实现热更新机制，也需要该方法
      function replaceReducer(nextReducer) {
        if (typeof nextReducer !== 'function') {
          throw new Error('Expected the nextReducer to be a function.')
        }
    
        currentReducer = nextReducer
        dispatch({ type: ActionTypes.REPLACE })
      }
      
      //
      function observable() {
        const outerSubscribe = subscribe
        return {
          subscribe(observer) {
            if (typeof observer !== 'object') {
              throw new TypeError('Expected the observer to be an object.')
            }
    
            function observeState() {
              if (observer.next) {
                observer.next(getState())
              }
            }
    
            observeState()
            const unsubscribe = outerSubscribe(observeState)
            return { unsubscribe }
          },
    
          [$$observable]() {
            return this
          }
        }
      }
    
      dispatch({ type: ActionTypes.INIT })
    
      return {
        dispatch,
        subscribe,
        getState,
        replaceReducer,
        [$$observable]: observable
      }
    }
```

这里最值得关注的就是currentListeners 和nextListeners这两个listener队列及ensureCanMutateNextListeners方法。每次修改的是nextListeners。每次flush listeners时 将nextListeners和currentListeners同步。

#### bindActionCreators.js

``` javascript  
    //bindActionCreators 用法
    bindActionCreators({ //第一个参数也可以是一个 actionCreator函数
        addCreator, //属性值是actionCreator函数
        anotherCreator,
        ...
    },dispatch)
    
    //bindActionCreator
    function bindActionCreator(actionCreator, dispatch) {
      return function() { 
        return dispatch(actionCreator.apply(this, arguments)) 
      }
    }
    
    //bindActionCreators返回值
    => boundActionCreators = {
        addCreator: bindActionCreator(addCreator, dispatch),
        anotherCreator: bindActionCreator(anotherCreator, dispatch),
        
    }
    or 
    => 
    
    bindActionCreator(actionCreators, dispatch) => function (...arg) {
        return dispatch(actionCreator(...arg))) 
    }
    
    //actionCreator
    actionCreator() => action (对象)，用的时候dispatch(actionCreator())
```
bindActionCreators 主要是把actionCreator 包了一层dispatch，使用时可以像普通函数一样调用actionCreator。使得对redux 无感知。

#### applyMiddleWare


``` javascript 

    import compose from './compose'
    
    export default function applyMiddleware(...middlewares) {
      return (createStore) => (...args) => {
        const store = createStore(...args)
        let dispatch = store.dispatch
        let chain = []
    
        const middlewareAPI = {
          getState: store.getState,
          dispatch: (...args) => dispatch(...args)
        }
        chain = middlewares.map(middleware => middleware(middlewareAPI))
        dispatch = compose(...chain)(store.dispatch)
    
        return {
          ...store,
          dispatch
        }
      }
    }

```


#### compose.js

``` javascript 

    export default function compose(...funcs) {
      if (funcs.length === 0) {
        return arg => arg
      }
    
      if (funcs.length === 1) {
        return funcs[0]
      }
    
      return funcs.reduce((a, b) => (...args) => a(b(...args)))
    }

```


### 参考资料
* [github](https://github.com/reactjs/redux)
* [中文文档](http://cn.redux.js.org/index.html)
* [源码解读](http://www.jianshu.com/p/7ae531a8b299?from=timeline)
* [极简教程](https://github.com/react-guide/redux-tutorial-cn#redux-tutorial)
* [极简代码示例]()


