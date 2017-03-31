---
title: Redux之中间件applyMiddleware源码解读
categories:
- 技术
- 源码
tags:
- react
- redux
---

在学习了redux过程中，了解到中间件这个名词，但是我看了十遍，也完全就是懵逼的状态。于是又重复敲了几次代码也不能掌握这个东西到底是什么？看官网文档，那么些专业名词，让人半天摸不着头脑。我们从流程图中知道，react组件在执行过程中，特别是在中间插入各种奇怪的需求的时候，不可能每每都是改动一下代码逻辑，而是用一种方便插拔的方式添加进去。但是这个过程说得简单，理解也容易，问题到底是怎么实现的呢？本人这样的半吊子水平，要理解这么高深的东西，真是太困难了。反复地摸索过程中，感觉摸到一些门径，于是斗胆做一下我的解读，如有不对，欢迎斧正！

### 前言
在学习appleyMiddleware中间件之前，必须要有一些知识储备。列出一下：
1. [react](http://reactjs.cn/)
1. [redux](http://cn.redux.js.org/index.html)或者看github上的教程[redux-tutorial-cn](https://github.com/react-guide/redux-tutorial-cn#redux-tutorial)


### 源码分析
#### applyMiddleware源码
```javascript
import compose from './compose'

export default function applyMiddleware(...middlewares) {
    return (createStore) => (reducer, preloadedState, enhancer) => {
        var store = createStore(reducer, preloadedState, enhancer)
        var dispatch = store.dispatch
        var chain = []

        var middlewareAPI = {
            getState: store.getState,
            dispatch: (action) => dispatch(action)
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

#### createStore
第一次执行applyMiddleware增加中间件，使用闭包保存中间件，然后返回一个函数（一开始我很奇怪为什么参数是createStore？？)，在弄明白applyMiddleware之前，得先来看他是如何被调用的，那就得先从createStore开始看。
摘了核心的`createStore`的源码如下：
```javascript
export default function createStore(reducer, preloadedState, enhancer) {
    if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
        enhancer = preloadedState
        preloadedState = undefined
    }

    if (typeof enhancer !== 'undefined') {
        if (typeof enhancer !== 'function') {
            throw new Error('Expected the enhancer to be a function.')
        }

        return enhancer(createStore)(reducer, preloadedState)
    }

    if (typeof reducer !== 'function') {
        throw new Error('Expected the reducer to be a function.')
    }
    ...
}
```
分析源码可以发现其中有一段这样的代码：
```javascript
if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
        throw new Error('Expected the enhancer to be a function.')
    }

    return enhancer(createStore)(reducer, preloadedState)
}
```
翻译一下这个参数`enhancer`英文意思为：“增强器”。
我反复看这代码，越看越吃惊，天啊！作者写的这段代码太让人惊叹了！我从未见过如此风骚的代码！

#### enhancer
我们不妨再简化一下这个`createStore`源码：
```javascript
function createStore(reducer,preloadedState,enhancer){
    ...
    return enhancer(createStore)(reducer,preloadedState)
    ...
}
```
我们知道，其实`enhancer === applyMiddleware`，这样子，我们再将`enhancer`换为`applyMiddleware`
这时变成这样子：
```javascript
function createStore(reducer,preloadedState,enhancer){
    ...
    return applyMiddleware(createStore)(reducer,preloadedState)
    ...
}
```
这时，我们可以将里面的返回代码拿出来，得出这样的：
```javascript
applyMiddleware(createStore)(reducer,preloadedState) -> <enhancer>
```
我们可以想一下，执行这段代码会返回什么呢？暂时先不用管，我们假设返回结果为<enhancer>。
从函数定义上，执行到此地，就被返回了，也就是到函数到此结束，下面的所有代码都不会执行了。那createStore函数，到这里，就交给了applyMiddleware去处理了。

#### applyMiddleware
我们再回到applyMiddleware的源码上来。看到，定义的applayMiddlware为（简化之后）：
```javascript
function applyMiddleware(...middlewares) {
    return (createStore) => (reducer, preloadedState, enhancer) => {
        // ...
    }
}
```
然后我们不妨一步一步调用此函数：
```javascript
// step1
const step1 = applyMiddleware(...middlewares) //返回 function(createStore){}

// step2
const step2 = step1(createStore) //返回 function(reducer,preloadedState,enhancer){}

// step2的返回值即为
const step3 = function(reducer,preloadedState,enhancer){}

// 对比createStore的定义
function createStore(reducer,preloadedState,enhancer){}
```
我们在此可以惊奇地发现，原来，createStore函数居然可以通过applyMiddleware返回的！！！那在此，可以得出，createStore(reducer,preloadedState,enhancer)执行之后，如果enhancer未传，那么就是普通的createStore了，如果传了，那实际上，createStore已经被enhancer接管了，然后相当于再返回一个普通的createStore而已。这才是其中的精妙之处！

我们再来看一下我们平时调用createStore时的方式是这样的：
```javascript
createStore(
    rootReducers,    //reducer
    preloadedState,
    applyMiddleware( //enhancer
        thunkMiddleware,
        createLogger
    )
)
```
可以将里面的`applyMiddleware`替换为`<enhancer>`，然后与上面的对比：
```javascript
createStore(reducer,preloadedState,<enhancer>)
// 实际上，enhancer传了参，那么返回的结果实际上也是一个普通的 createStore
```
在第一次调用createStore的时候，createStore先判断是否有middlewares(enhancer)的加入，如果有，就不执行createStore后面的操作，return出去执行enhancer()。这里换一种说法：

> 执行createStore的时候，只要传了中间件applyMiddleware这样的合法参数，那么，就相当于createStore被改写了，实际返回时，也是一个createStore方法，然后执行之后，与普通的是一样的，而且中间可以随意添加移除各种需求的逻辑组件。此种实现方法被冠以一个名词：[柯里化(Currying)](https://en.wikipedia.org/wiki/Currying)，就是将多参变成单参的函数，也就是通过函数链的方式进行返回以达到单参函数。这就是applyMiddleware中间件的核心价值！

** 注意：执行了enhancer(createStore)后，只传入两个参数(reducer,preloadedState)，第三个参数 enhancer为undefined **

#### store
执行enhancer就要回过头看applyMiddleware源码。
实际上执行enhancer，返回就是我们要的createStore函数！
```javascript
function applyMiddleware(...middlewares) {
    return (createStore) => (reducer, preloadedState, enhancer) => {
        // ...
        return {
          ...store,
          dispatch
        }
    }
}
//执行之后，返回值为createStore函数：
function(reducer, preloadedState, enhancer){}
```
由于没有第三个参数enhancer，所以这才是真正执行createStore()，返回一个没有 middleware的store。
我们可以看一下applyMiddleware里面有一个语句：
```javascript
...
var store = createStore(reducer, preloadedState, enhancer)
...
```
在这里，可以看出，在内部，也执行了createStore函数的调用，也就是说，createStore将实现移交给了applyMiddleware之后，在applyMiddleware内部同样会生成普通的store对象的。同样，如果这里的enhancer如果存在，继续循环原先的步骤。

#### middleware
我们继续看内部的源码为：
```javascript
...
var dispatch = store.dispatch
var chain = []

var middlewareAPI = {
  getState: store.getState,
  dispatch: (action) => dispatch(action)
}
chain = middlewares.map(middleware => middleware(middlewareAPI))
...
```
定义了一个chain数组，存放每一个被处理过的middleware。
代码可以这样解释：首先为每一个middleware以`{getState，dispatch}`为参数执行一遍，其实是为了给middleware一个原生的`{getState，dispatch}`两个方法的指针，以便在middleware中调用。
而上面的applyMiddleware的参数中就有两个参数：**thunkMiddleware**,**createLogger**，他们都是middleware。他们在传入applyMiddleware的过程中，都被包装过一次，并且存放在chain数组中。
请看一个简单的middleware：
```javascript
const logger = ({getState,dispatch}) => next => action {
    console.log('dispatching',action)
    let result = next(action)
    console.log('next state',getState())
    return result
}
```
调用后返回的chain是一个以next为参数的函数数组:
```javascript
chain = [logger].map(middleware => middleware({
    getState:store.getState,
    dispatch:(action) => dispatch(action)
}))
```

#### compose
继续看代码，有一个这样的语句：
```javascript
...
dispatch = compose(...chain)(store.dispatch)
...
```
dispatch被compose包装之后，重新赋值给自身。但这段语句看得莫名莫妙。这是什么鬼意思？干嘛用的？

这里，不妨来看一下compose源码
```javascript
export default function compose(...funcs) {
    if (funcs.length === 0) {
        return arg => arg
    }

    if (funcs.length === 1) {
        return funcs[0]
    }

    const last = funcs[funcs.length - 1]
    const rest = funcs.slice(0, -1)
    return (...args) => rest.reduceRight((composed, f) => f(composed), last(...args))
}
```
其中一段为：
```javascript
return (...args) => rest.reduceRight((composed, f) => f(composed), last(...args))
```
看到这段，更加头疼。。。反正看来看去。大体意思是这样：`compose` 可以接受一组函数参数，从右到左来组合多个函数，然后返回一个组合函数。
也就是说，compose接收chain这个数组，然后用reduceRight函数进行组合，最终合成了一个新的函数，只能这么理解了。

#### dispatch
到这里，也就是说，dispatch已经被compose重新组装过一次，在最后，再被组装成一个新的store返回。
```javascript
...
return {
  ...store,
  dispatch
}
```

### 结论
middleware内部的dispatch是原生的没有middleware时的dispatch，
每一个middleware都带有原生的getState，dispatch和next（下一个middleware），所以我可以在middleware中不调用next，而直接调用dispatch，就跳过了后面的middleware了。
applyMiddleware中间件，其实就是将createStore接管了，然后在最终返回一个store对象。每一个中间件都是做这样的一件事情，这样，就可以源源不断地往里面添加需求或者移出需求，而不必修改流程代码上的任何逻辑。仅此而已！