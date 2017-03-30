---
title: applyMiddleware源码解读
categories:
- 技术
- 源码
tags:
- react
- redux
---
在学习了redux过程中，了解到中间件这个名词，但是我看了十遍，也完全就是懵逼的状态。于是又重复敲了几次代码也不能掌握这个东西到底是什么？有什么用处。在此尝试性的总结一番。

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

#### step1 applyMiddleware(...middlewares)
第一次执行applyMiddleware增加中间件，使用闭包保存中间件，然后返回一个函数（一开始我很奇怪为什么参数是createStore？？)，在弄明白applyMiddleware之前，得先来看他是如何被调用的，那就得先从createStore开始看。

#### step2 createStore源码摘要
为什么参数是`createStore`？我看了createStore源码就知道了原因。
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
分析源码可以发现其中有一段
```javascript
if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
        throw new Error('Expected the enhancer to be a function.')
    }

    return enhancer(createStore)(reducer, preloadedState)
}
```
翻译一下这个参数`enhancer`英文意思为：“增强子”。
再来看一下我们平时调用createStore时的方式：
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
在第一次调用createStore的时候，createStore先判断是否有middlewares(enhancer)的加入，如果有，就不执行createStore后面的操作，return出去执行enhancer()
** 注意：执行了enhancer(createStore)后，只传入两个参数(reducer,preloadedState)，第三个参数 enhancer为undefined **

#### step3 enhancer()
执行 enhancer 就要回过头看applyMiddleware源码。
由于没有第三个参数enhancer，所以这才是真正执行 createStore(), 返回一个没有 middleware 的 store。
#### step4 middleware
首先为每一个middleware以`{getState，dispatch}`为参数执行一遍，其实是为了给middleware一个原生的`{getState，dispatch}`两个方法的指针。以便在middleware中调用。
请看一个简单的middleware：
```javascript
const logger = ({getState,dispatch}) => next => action {
    console.log('dispatching',action)
    let result = next(action)
    console.log('next state',getState())
    return result
}
```
调用后返回的chain是一个以next为参数的函数数组。

#### step5 compose
** _dispatch = _compose2['default'].apply(undefined, chain)(store.dispatch) **
看一下compose源码
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
得出`arguments === store.dispatch`
因此compose后返回的_dispatch是多个middleWares嵌套而成的函数，每一个next闭包变量都是里层的middleware，并且最终的next是store.dispatch

#### step last
用新的middleware多层嵌套的_dispatch代替store.dispatch，就over了

#### 结论
middleware内部的dispatch是原生的没有middleware时的dispatch，
每一个middleware都带有原生的getState，dispatch和next（下一个middleware），所以我可以在middleware中不调用next，而直接调用dispatch，就跳过了后面的middleware了。