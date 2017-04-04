---
title: 从实际应用中去理解Promise
categories:
- 技术
- 应用
tags:
- es6
- promise
- 原生对象
---

在es6使用已经很普遍的今天，我再说Promise各种API及其原理的话也不过是锦上添花。Promise的重要性，我们不必多讲，相信大家都知道出现Promise的意义及其对js异步编程上的作用是多么巨大。不过，说真，要真正去理解使用这个对象，确实是有些困难，特别是对我这种理解新东西总是有困难症的人来说更甚。所以，我的办法就是重重复复地看相关的文章及api文档，以达到自己能使用及理解。在理解Promise之前，先来看一下异步的实现过程。

### 为什么要用Promise
在说Promise之前，我们得先来确定一下为什么要有这个对象。
#### 原生JS实现一个简单的ajax
说到浏览器异步处理中，ajax可以说是最基本的异步编程方法了，可以简单实现为：
```javascript
//此实现，只是简单实现，未实现兼容处理
var url = '<url>';
var result;

var XHR = new XMLHttpRequest();
XHR.open('GET', url, true);
XHR.send();

XHR.onreadystatechange = function() {
    if (XHR.readyState == 4 && XHR.status == 200) {
        result = XHR.response;
        console.log(result);
    }
}
```
在ajax的原生实现中，利用了`onreadystatechange`事件，当该事件触发且符合一定条件时，才能拿到我们想要的数据。之后我们才能开始执行回调里面的代码。这看上去并没有什么麻烦的，但是如果这时，我们想有两个ajax请求，并且还得有先后顺序进执行。我们这时会这样做。
```javascript
var url = '<url>';
var result;

var XHR = new XMLHttpRequest();
XHR.open('GET', url, true);
XHR.send();

XHR.onreadystatechange = function() {
    if (XHR.readyState == 4 && XHR.status == 200) {
        result = XHR.response;
        console.log(result);
        // 在此重复上面的逻辑
        var url2 = '<url>';
        var result2;
        var XHR2 = new XMLHttpRequest();
        XHR2.open('GET', url, true);
        XHR2.send();
        XHR2.onreadystatechange = function() {
            if (XHR.readyState == 4 && XHR.status == 200) {
                result = XHR.response;
                console.log(result);
            }
        }
    }
}
```
当出现第三个ajax（甚至更多）仍然依赖上面的方法做，那将是一场灾难。这样的灾难，往往被称为** 回调地狱 **
因此，我们需要消除掉回调地狱这个问题。
当然，除了消除回调地狱之外，还一个非常重要的需求：** 为了我们的代码更加具有可读性与可维护性，我们需要将数据请求与数据处理逻辑区分开来 **。上面的写法，是完全没有区分开的，当数据变得复杂，处理逻辑就更加复杂，到时我们就没法维护我们写的代码了。
（为了代码简洁性，部分代码使用es6的写法）
#### 函数调用栈
我们可以通过利用函数调用栈，将我们想要代码放到回调函数中，来解决**回调地狱**问题。函数调用栈这是一种什么概念呢？我们知道，js内置了一些公共的方法，如setTimeout与setInterval这两个函数，他们有一个特性，就是异步执行。当js执行到此类函数的时候，直接放到栈中（具体如何实现不做详解），当整个同步的js代码执行完成之后，再调用栈中的异步函数，当然这个调用栈也是按放入的顺序去一个一个执行。这样子我们就实现了异步处理按我们想要队列办法来实现。
```javascript
function fn(){
    console.log('fn do!');
}
function fn1(fn){
    console.log('do 1!');
    fn && setTimeout(fn,0);
}
function fn2(fn){
    console.log('do 2!');
    fn && setTimeout(fn,0);
}
function fn3(fn){
    console.log('do 3!');
    fn && setTimeout(fn,0);
}
fn1(fn2(fn3(fn)));
// 输出结果为：
// do 3!
// do 2!
// do 1!
// fn do!
```
这种实现办法，虽然在一定程序上可以解决掉代码维护上的困难度，不过却影响到可读性。我们从调用的层叠性上看，令人搞不清顺序，从fn1到fn3的层层调用中，我们发现输出结果是先3然后再到1，最后才执行want函数，感觉虽然解决了代码回调地狱，却把顺序弄得不伦不类的。如果这队列上的回调更多，则会更加难以理解。

### Promise的基础与使用
#### Promise的引入
如果浏览器已经支持了原生的Promise对象，那我们可以将上面的函数都改写为Promse对象的方法。
```javascript
function fn(){
    console.log('fn do!');
}
function fn1(fn){
    console.log('do 1!');
    return new Promise(function(resolve,reject){
        if(typeof fn == 'function') resolve(fn);
        else reject("TypeError:" + fn + "no a function.");
    });
}
function fn2(fn){
    console.log('do 2!');
    return new Promise(function(resolve,reject){
        if(typeof fn == 'function') resolve(fn);
        else reject("TypeError:" + fn + "no a function.");
    });
}
function fn3(fn){
    console.log('do 3!');
    return new Promise(function(resolve,reject){
        if(typeof fn == 'function') resolve(fn);
        else reject("TypeError:" + fn + "no a function.");
    });
}
fn1(fn)
.then(function(fn){
    return fn2(fn);
})
.then(function(fn){
    return fn3(fn);
})
.then(function(fn){
    fn();
});
// 执行结果为：
// do 1!
// do 2!
// do 3!
// fn do!
```
从上面的代码中，我们很清楚地看到，fn1是从1-3的顺序去执行，并且代码层次也可以分得非常清晰。这就是Promise的在实际中最基本的实现。

#### Promise对象
为了更好理解Promise，我们把其基础的概念进行解释一下：
一、Promise对象有三种状态，分别是：
* padding:等待中，或进行中，表示还没有得到结果
* resolved(Fulfilled): 已经完成，表示得到了我们想要的结果，可以继续往下执行
* rejected: 也表示得到结果，但是由于结果并非我们所愿，因此拒绝执行

```javascript
new Promise(function(resolve, reject) {
    if(true){ resolve() };
    if(false){ reject() };
})
```
二、Promise对象中的then方法，可以接收构造函数中处可以接收构造函数中处理的状态变化，并分别对应执行。then方法有2个参数，第一个函数接收resolved状态的执行，第二个参数接收reject状态的执行。
> 假设：var p = new Promise(resolve,reject)中有两参数，则p.then((data1) => {},(data2) => {})，此中data1是resolve执行处理的结果，而data2是reject执行处理的结果。

then方法也会返回一个Promise对象。因此我们就可以进行then的链式调用了。这也是解决回调的主要方式。比如可以这样写：
```javascript
new Promise((resolve,reject) => {
    if(true){
        console.log('resolve');
        resolve();
    }else{
        console.log('reject');
        reject();
    }
}).then(() => {
    console.log('then 1');
}).then(() => {
    console.log('then 2');
})
```
我们在浏览器控制台上打印`new Promise((resolve,reject) => {})`，从原型中可以看到除了then还有一个catch。
而`then(null,() => {})`就等同于`catch(() => {})`。从这个结构上来看，then函数其实也是接受两个参数的。我们可以猜想，在执行异步的队列过程中，每一步都是有返回成功与失败的情况，那么则在每一种情况下都可以执行相应的逻辑代码。
```javascript
// 伪代码
new Promise((resolve,reject) => {})
    .then(() => {}, () => {}) //第一个回调是成功的，第二个是失败的
    .then(() => {}, () => {})
    ...
```
或者根据Promise的对象实例接口可以分开来写：
```javascript
// 伪代码
new Promise((resolve,reject) => {})
    .then(() => {}) //成功的
    .catch(() => {}) //失败的
    .then(() => {})
    .catch(() => {})
    ...
```
三、Promise.all方法是等待所有Promise都执行完之后才执行的回调函数。比如，很多ajax在被按顺序地执行，当所有ajax都返回值了，这时才去执行all。
Promise.all接收一个Promise对象组成的数组作为参数，当这个数组所有的Promise对象状态都变成resolved或者rejected的时候，它才会去调用then方法。并且返回一个Promise对象。
```javascript
const p1 = new Promise((resolve,reject) => {});
const p2 = new Promise((resolve,reject) => {});
const p3 = new Promise((resolve,reject) => {});
Promise.all([p1,p2,p3]).then(() => {
    console.log('run Promise all callback!');
})
```
四、Promise.race方法则是在一个Promise对象数组中，只要有一个Promise的状态变成resolved或者rejected，也就是说只要有一个执行完成，就可以调用其then方法了。
同样Promise.race与是接收一个Promise数组作为参数，并返回一个Promise对象。
```javascript
const p1 = new Promise((resolve,reject) => {});
const p2 = new Promise((resolve,reject) => {});
const p3 = new Promise((resolve,reject) => {});
Promise.race([p1,p2,p3]).then(() => {
    console.log('run Promise all callback!');
})
```

#### Promise中的数据传递
Promise的then会执行之后会返回一个Promise对象，而then方法的两个可选参数是两个回调函数，这两个回调函数都有一个可选参数，实际上，这个参数就是从上一个Promise处理后resolved或者rejected的结果值。
```javascript
new Promise(function(resolve,reject) {
    if(true) resolve('resolve 0');
    else reject('reject 0');
}).then(function(data){
    console.log(data);
    return "resolve 1";
},function(data){
    console.log(data);
    return "reject 1";
}).then(function(data){
    console.log(data);
    return "resolve 2";
},function(data){
    console.log(data);
    return "reject 2";
}).then(function(data){
    console.log(data);
    return "resolve 3";
},function(data){
    console.log(data);
    return "reject 3";
}).then(function(data){
    console.log(data);
},function(data){
    console.log(data);
});

// 打印的结果为：
// resolve 0
// resolve 1
// resolve 2
// resolve 3
```
从上面的结果来看，可以知道，每一个then方法的返回值，都是下一then方法中的回调函数中的参数值。这样内中的值就可以按着我们想要的顺序一步一步地向下传。

### Promise在实际中的应用
Promise的实际应用是非常广泛的，常见的异步编程中，基本上都可以使用Promise来实现。
#### 应用Promise封装ajax
在文章开始的时候，我们做过一个简便的ajax实际，现在我们再利用Promise来实现其过程。
```javascript
// 简略实现ajax
function ajax(url,type){
    return new Promise((resolve,reject) => {
        var XHR = new XHRHttpRequest();
        XHR.open(type,url,true);
        XHR.send();
        XHR.onreadystatechange = () => {
            if(XHR.readyState == 4) {
                if(XHR.status == 200) {
                    try {
                        let response = JSON.parse(XHR.responseText);
                        resolve(response);
                    } catch(e) {
                        reject(e);
                    }
                } else {
                    reject(new Error(XHR.statusText));
                }
            }
        }
    });
}
```
为了健壮性，处理了很多可能出现的异常，总之，就是正确的返回结果，就resolve一下，错误的返回结果，就reject一下。并且利用上面的参数传递的方式，将正确结果或者错误信息通过他们的参数传递出来。
然后在调用的时候，可以这样使用：
```javascript
// 伪代码
ajax(<url>,<type>).then(response => console.log(response), error => console.log(error));
```
> 现在所有的库几乎都将ajax请求利用Promise进行了封装，因此我们在使用jQuery等库中的ajax请求时，都可以利用Promise来让我们的代码更加优雅和简单。这也是Promise最常用的一个场景，因此我们一定要非常非常熟悉它，这样才能在应用的时候更加灵活。

所以现在的jquery中的$.ajax实际上是可以这样子调用的：
```javascript
$.ajax(<url>).done(callback).error(callback);
```

### 总结
Promise在实际应用中是非常广泛的，常见的ajax封装，图片异步加载，一些js插件的封装也可以通过异步的实现办法。真正去理解它，还需要不段地练习以相多多查看api文档。这样才能更好利用Promise为你做更多的事。
写此文我参考了一些别人的资料，并添加了一些自己的见解，七凑八凑而成。由于我的知识积累有限，有很多地方可能解释得也是不太清楚。在此给一下参考作者的文章：[前端进阶：透彻掌握 Promise 的使用，读这篇就够了](https://juejin.im/entry/58e1d720ac502e006c0e0196?from=singlemessage&isappinstalled=1)