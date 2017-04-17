---
title: React全家桶之初体验
categories:
- 技术
- 总结
tags:
- es6
- react
- redux
- immutable
date: 2017/4/15
---

接触React也有段时间了，但是从初学者的角度来看，官网的资料实在是少，看起来比较吃力，并且是英文的，而那中文的网站却也是翻译的乱七八糟的，无力吐槽。我作为一个初学者，我谈不上有什么高深见解，偶尔有一些心得与体会，将之分享给与我一样的刚入门的同学，如果您已经比较熟悉这些知识了，请忽视此篇。另外我想说明的是，此文不是教程或者文档，关于理解也不够深刻，有偏差或者误解的地方欢迎指正。下面是从我的角度去解释如何实现一些简单的例子。如果有些概念不理解可以看[React学习资源汇总](https://github.com/tsrot/study-notes/blob/master/React%E5%AD%A6%E4%B9%A0%E8%B5%84%E6%BA%90%E6%B1%87%E6%80%BB.md)，这里有详细的文档教大家如何去学习React。
因为facebook已经完全掏拥抱ES6标准，所以我们也遵循官网的标准进行编写我们的代码。我是以案例去说明这个react全家桶（我在此案例代码编辑的过程中，不在乎文件夹的划分，全部代码写在一个文件里面，实际开发中请注意区分）。

### React全家桶成员及初步项目搭建
#### 关于全家桶的理解
大家都知道react.js仅仅是一个UI库，说白了，它只关心views的渲染，其它的都不管。为了做出大型web应用，他还需要其它插件帮忙才行。
React全家桶里有非常多的技术及内容要学。在此讲不完所有的成员，因为根据项目需要，会有增减。所以，我只从核心组成方面讲几个，以达到学习react的组合开发的效果。
我列举来讲的全家桶是：**ES6 + react + redux + immutable.js + webpack + yarn**

#### 搭建项目
利用yarn（也可以使用npm）包管理工具搭建我们的项目，前提是必须先安装yarn。
```bash
$ npm i -g yarn-cli
```
安装完yarn之后，利用yarn进行初始化项目
```bash
$ yarn init -y
```
安装包依赖，也就是我们的核心的技术包支持
```bashk
$ yarn add react react-dom babel babel-core babel-loader babel-preset-es2015 babel-preset-stage-0 babel-preset-react webpack webpack-dev-server
```
安装完所有的依赖包，在package.json文件中再添加一个配置项，用于热布署与实时监测修改自动刷新浏览器：
```json
"scripts": {
    "watch": "webpack-dev-server --port 8080 --colors"
}
```
则此package.json文件为：
```json
{
  "name": "react-family",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "dependencies": {
    "babel": "^6.23.0",
    "babel-core": "^6.24.1",
    "babel-loader": "^6.4.1",
    "babel-preset-es2015": "^6.24.1",
    "babel-preset-react": "^6.24.1",
    "babel-preset-stage-0": "^6.24.1",
    "react": "^15.5.4",
    "react-dom": "^15.5.4",
    "webpack": "^2.4.1",
    "webpack-dev-server": "^2.4.2"
  },
  "scripts": {
    "watch": "webpack-dev-server --port 8080 --colors"
  }
}

```
然后添加一个文件webpack.config.js，具体怎么进行各项配置在此不做过多解释，直接上代码：
```javascript
var webpack = require('webpack');

module.exports = {
    entry: {
        index: ['./index.js'],
        vendor: [
            'react',
            'react-dom'
        ]
    },
    output: {
        filename: '[name].bundle.js',
        path: __dirname + '/',
        publicPath: '/public'
    },
    module: {
        loaders: [{
            test: /\.js$/,
            exclude: /node_modules/,
            loader: 'babel-loader',
            query: {
                presets: ['react','es2015','stage-0']
            }
        }]
    },
    resolve: {
        extensions: [' ', '.js', '.json', '.scss']
    },
    plugins: [
        new webpack.optimize.CommonsChunkPlugin({name: 'vendor',filename:'vendor.bundle.js'})
    ]
}
```
OK，我们的项目脚手架已经搭建完成。接下来就可以开始coding了。
添加一个入口文件index.html则可进行我们react的开发。
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>react-family</title>
</head>
<body>
    <div id="root"></div>
    <script type="text/javascript" src="/public/vendor.bundle.js"></script>
    <script type="text/javascript" src="/public/index.bundle.js"></script>
</body>
</html>
```

### React组件
#### 需求一
请根据如下需求实现一个计数器：
1. 利用reactjs实现一个加减计器Couter
2. 点“+”加1，点“-”减1

#### 基本形式
因为需求非常简单，我们可以只利用react的一个单独的组件就可以完成这个需求。
```javascript
import React, { Component } from 'react';
import { render } from 'react-dom';

class Counter extends Component {
    constructor(props) {
        super(props);
        this.state = {
            count: 0
        }
    }
    handleSub(number) {
        this.setState({count: --number });
    }
    handleAdd(number) {
        this.setState({count: ++number })
    }
    render() {
        let count = this.state.count;
        let input;
        return (
            <div>
                <input type="button" value="-" onClick={() => this.handleSub(input.value)} />
                <input type="text" value={count} style={{width: "50px", textAlign: "center"}} ref={node => {input = node}} />
                <input type="button" value="+" onClick={() => this.handleAdd(input.value)} />
            </div>
        )
    }
}

render(
    <Counter />,
    document.getElementById('root')
)
```
利用React的状态state可变的特性，可以持续修改状态值，并实时更新UI。
#### 精简形式
可以将代码写得更精简一些：
```javascript
...
class Counter extends Component {
    constructor(props) {
        super(props);
        this.state = {
            count: 0
        }
    }
    render() {
        let count = this.state.count;
        return (
            <div>
                <input type="button" value="-" onClick={() => this.setState({count: --count})} />
                <input type="text" value={count} style={{width: "50px", textAlign: "center"}} />
                <input type="button" value="+" onClick={() => this.setState({count: ++count})} />
            </div>
        )
    }
}
...
```
我们的核心就是通过更新state来改变数据状态。这样子，我们就可以直接去掉ref属性，我们不需要拿到数据，只需要在state的原基础之上进行修改就可以达到数据的更新。而事件绑定，我们更不需要另外多写更多逻辑，直接利用es6的方式就可以进行快速实现。

#### 无状态组件
实际开发中，我们写的React组件都是分成两种形式，一种是纯组件，叫**无状态组件**，只用于显示，不做数据操作逻辑，一般存放在components目录。另一种是就是数据操作组件，是将数据与无状态组件绑定在一起连接的组件，一般放在containers目录。当然我们因为例子太简单，所以不进行目录划分，但要分开来写。
上面的需求，我们完全可以利用无状态组件进行操作。划分为逻辑组件与无状态组件。便于开发维护，抽出来的无状态组件为：
```javascript
const CounterDOM = ({ count, sub, add }) => (
    <div>
        <input type="button" value="-" onClick={sub} />
        <input type="text" value={count} style={{width: "50px", textAlign: "center"}} />
        <input type="button" value="+" onClick={add} />
    </div>
)
```
而得到的数据组件为：
```javascript
class Counter extends Component {
    constructor(props) {
        super(props);
        this.state = {
            count: 0
        }
    }
    render() {
        let count = this.state.count;
        return <CounterDOM
                count={count}
                sub={() => this.setState({count: --count})}
                add={() => this.setState({count: ++count})} />
    }
}
```
在维护组件的时候，如果做显示修改，我们只要修改显示组件，如果做数据更新只做数据组件的更改。

> 上面的例子中涉及到父子组件之间进行通信，而子组件实际上只负责显示，则在子组件只取props组件进行渲染，些案件中`({ count, sub, add })`中的内容，里面的对象即为props的**解构过程**，而做交互、做数据修改则在父组件上做更新。“无状态组件没有实例”，即不能使用任何诸如 componentWillMount 或 shouldComponentUpdate 的生命周期方法。

### Redux架构
我们知道，组件分为数据与显示，那么我们应该把数据的逻辑也分开来。官方推荐使用redux进行数据层的处理。我们根据上面的需求，使用redux构架进行拆分与组装，将数据层的结构交给redux去管理，让react负责渲染层。这样子可以让显示与数据再加的松藕合了。
> 我们在使用redux的时候，你可能不需要它，Redux 是一个有用的架构，但不是非用不可。事实上，大多数情况，你可以不用它，只用 React 就够了。**"只有遇到 React 实在解决不了的问题，你才需要 Redux 。"**

使用之前得先安装redux：
```javascript
$ yarn add redux
```
使用redux必须要理解的概念：**action**, **reducer**, **store**, **dispatch**, **subscribe**。关于redux的资料可以看[redux中文教程](http://www.redux.org.cn/)。

#### createStore
而redux提供了一个createStore方法，用于创建一个唯一的store树，是整个应用的数据仓库。我们就可以将整个应用的数据结构与处理过程交由store来管理。
先来尝试一下redux：
```javascript
import { createStore } from 'redux';

// Actions
const SUB = 'SUB';
const sub = () => ({type: SUB});
const ADD = 'ADD';
const add = () => ({type: ADD});

// Reducers
const counter = (state = 0, action) => {
    switch(action.type) {
        case SUB: return state - 1;
        case ADD: return state + 1;
        default: return state;
    }
}

const store = createStore(counter);
store.dispatch(sub())
console.log(store.getState())
// -1
```
我们知道state数据是被存在一个叫做store的对象中。而store是唯一的，所有的state对象都是经由这个store进行处理之后，再返回一个新的state。
而引发更新这一事件过程的对象就叫做action。由它触发事件之后，交给纯函数reducers处理之后返回一个新的state对象。
我们学会了使用 action 来描述“发生了什么”，和使用 reducers 来根据 action 更新 state 的用法。
store就是把它们联系到一起的对象。Store 有以下职责：
1. 维持应用的 state；
1. 提供 getState() 方法获取 state；
1. 提供 dispatch(action) 方法更新 state；
1. 通过 subscribe(listener) 注册监听器;
1. 通过 subscribe(listener) 返回的函数注销监听器。

如此，我们结合到显示组件上去，就实现了显示与数据分离管理。

```javascript
import React, { Component } from 'react';
import { render } from 'react-dom';
import { createStore } from 'redux';

// Actions
const SUB = 'SUB';
const sub = () => ({type: SUB});
const ADD = 'ADD';
const add = () => ({type: ADD});

// Reducers
const counter = (state = 0, action) => {
    switch(action.type) {
        case SUB: return state - 1;
        case ADD: return state + 1;
        default: return state;
    }
}

// store
const store = createStore(counter);

// conponents
const CounterDOM = ({ count, sub, add }) => (
    <div>
        <input type="button" value="-" onClick={sub} />
        <input type="text" value={count} style={{width: "50px", textAlign: "center"}} />
        <input type="button" value="+" onClick={add} />
    </div>
)
// containers
class Counter extends Component {
    render() {
        let count = store.getState();
        return (
            <CounterDOM
                count={count}
                sub={() => store.dispatch(sub())}
                add={() => store.dispatch(add())}
            />
        )
    }
}
// render
const renderDOM = () => render(
    <Counter />,
    document.getElementById('root')
)

renderDOM();
store.subscribe(renderDOM);
```
如此，我们的state就不是直接在原来的基础之上修改了。而是交给dispatch发起一个action，由reducer监听到action触发的事件就运算处理，返回一个新的state，再由react组件中的自动state自动渲染机制更新UI。整个过程都是数据单向流动的，过程中如下：
**views -> action -> reducer -> views**
我们觉得上面的做法还是有些麻烦，虽然我们把数据与显示分开了，但是在数据组件的时候，还要传很多参数，并且在组件里面直接调用store对象进行数据操作了。虽然react与redux结合起来使用了，但还需要一些额外的工具，其中最重要的莫过于react-redux了。

#### Provider、connect
react-redux 提供了两个重要的对象，**Provider**和**connect**，前者使 React 组件可被连接（connectable），后者把 React 组件和 Redux 的 store 真正连接起来。
```javascript
import React, { Component } from 'react';
import { render } from 'react-dom';
import { createStore } from 'redux';
import { Provider, connect } from 'react-redux';

// Actions
const SUB = 'SUB';
const sub = () => ({type: SUB});
const ADD = 'ADD';
const add = () => ({type: ADD});

// Reducers
const counter = (state = 0, action) => {
    switch(action.type) {
        case SUB: return state - 1;
        case ADD: return state + 1;
        default: return state;
    }
}

// store
const store = createStore(counter);

// conponents
const CounterDOM = ({ count, sub, add }) => (
    <div>
        <input type="button" value="-" onClick={sub} />
        <input type="text" value={count} style={{width: "50px", textAlign: "center"}} />
        <input type="button" value="+" onClick={add} />
    </div>
)

// containers
const Counter = connect(
    state => ({count: state}),
    dispatch => {
        return {
            sub: () => dispatch(sub()),
            add: () => dispatch(add())
        }
    }
)(CounterDOM)

// render
render(
    <Provider store={store}>
        <Counter />
    </Provider>,
    document.getElementById('root')
)
```
我们可以利用Provider将组件包起来。然后利用connect将其与数据层连接在一起。这样子。我们就不需要在组件中定义与编写我们的发布与订阅的监听了。然后，我们可以将这些逻辑写到我们的mapDispatchToProps函数中去。而我们的获取到的初始化值可以在mapStateToProps函数中写，返回一个纯js对象。这两个函数可以当作参数传给connect复合函数进行连接。更利于连接react与redux的便捷，也解决了赋值的方便性。这种做法的好处不言而喻了。

#### combineReducers、bindActionCreators
假设，产品并不满足现在的需求。他想在的当前的基础之上再添加一个功能。
1. 添加一个输入框，用户只能输入数字，作为商品单价；
1. 点击计数器的时候，自动计算数量与单位的积；
1. 添加一个显示积的标签用于显示积的值。

这时我们来添加一个action用于输入单价并且再添加一个reducer用于处理修改单价之后的积。我们现在有多个reducer，这时我们就必须将这些reducers都合并在一起，使用一个combineReducers的函数进行整合。
```javascript
import React, { Component } from 'react';
import { render } from 'react-dom';
import { createStore, combineReducers } from 'redux';
import { Provider, connect } from 'react-redux';

// Actions
const SUB = 'SUB';
const sub = () => ({type: SUB});
const ADD = 'ADD';
const add = () => ({type: ADD});

const PRICE = 'PRICE';
const inputPrice = price => ({type: PRICE, price});

// Reducers
const counter = (state = 0, action) => {
    switch(action.type) {
        case SUB: return state - 1;
        case ADD: return state + 1;
        default: return state;
    }
}

const price = (state = 1, action) => {
    switch(action.type) {
        case PRICE: return action.price;
        default: return state;
    }
}

// store
const store = createStore(combineReducers({counter, price}));

// conponents
const DOM = ({ counter, sub, add, price, result, inputPrice }) => (
    <div>
        <div>
            <input type="button" value="-" onClick={sub} />
            <input type="text" value={counter} style={{width: "50px", textAlign: "center"}} />
            <input type="button" value="+" onClick={add}/>
        </div>
        <input type="text" value={price} style={{width: "94px"}} onChange={e => inputPrice(e.target.value)} />
        <p>{result}</p>
    </div>
)

// containers
const Counter = connect(
    state => ({
        ...state,
        result: state.counter * state.price
    }),
    dispatch => {
        return {
            sub: () => dispatch(sub()),
            add: () => dispatch(add()),
            inputPrice: price => dispatch(inputPrice(price))
        }
    }
)(DOM)

// render
render(
    <Provider store={store}>
        <Counter />
    </Provider>,
    document.getElementById('root')
)
```
而上面的写法还可以将actions都合并在一起，包装成一个个被dispatch处理的actions。
```javascript
...
// containers
const Counter = connect(
    state => ({
        ...state,
        result: state.counter * state.price
    }),
    dispatch => bindActionCreators({sub,add,inputPrice}, dispatch)
)(DOM)
...
```

#### @connect指令
我们知道，上面的写法虽然很直白了。不过我们还可以省掉一步，将定义的组件传入指令@connect，作为一个参数。这样指令在执行的过程就自动将紧接着的一组件包装起来，将数据与其联系在一起，然后返回一个新的类（组件）。这样做更形象一些，并且达到了更松的藕合度。
使用之前别忘了先安装插件babel-plugin-transform-decorators-legacy
```bash
$ yarn add babel-plugin-transform-decorators-legacy
```
然后在webpack.config.js中添加一个配置项：
```javascript
...
query: {
    presets: ['react','es2015','stage-0'],
    plugins: ['transform-decorators-legacy']
}
...
```
如此，代码可以这样写：
```javascript
import React, { Component } from 'react';
import { render } from 'react-dom';
import { createStore, combineReducers, bindActionCreators } from 'redux';
import { Provider, connect } from 'react-redux';

// Actions
const SUB = 'SUB';
const sub = () => ({type: SUB});
const ADD = 'ADD';
const add = () => ({type: ADD});

const PRICE = 'PRICE';
const inputPrice = price => ({type: PRICE, price});

// Reducers
const counter = (state = 0, action) => {
    switch(action.type) {
        case SUB: return state - 1;
        case ADD: return state + 1;
        default: return state;
    }
}

const price = (state = 1, action) => {
    switch(action.type) {
        case PRICE: return action.price;
        default: return state;
    }
}

// store
const store = createStore(combineReducers({counter, price}));

// containers
@connect(
    state => ({
        ...state,
        result: state.counter * state.price
    }),
    dispatch => bindActionCreators({sub,add,inputPrice}, dispatch)
)
class Counter extends Component {
    render() {
        const { counter,price,result,sub,add,inputPrice } = this.props;
        return (
            <div>
                <div>
                    <input type="button" value="-" onClick={sub} />
                    <input type="text" value={counter} style={{width: "50px", textAlign: "center"}} />
                    <input type="button" value="+" onClick={add}/>
                </div>
                <input type="text" value={price} style={{width: "94px"}} onChange={e => inputPrice(e.target.value)} />
                <p>{result}</p>
            </div>
        )
    }
}

// render
render(
    <Provider store={store}>
        <Counter />
    </Provider>,
    document.getElementById('root')
)
```
> 注意，这里的@connect()指令，必须写在react组件之前，紧接着组件。@connect指令实际上等价于connect(mapStateToPropes,mapDispatchToProps)(DOM)这种调用方式。

#### 中间件及包装函数applyMiddleware
我们的应用经过层层包装，从维护与可读性上都已经提高了好多。不过现在产品又加了个需求，就是监测中间的数据的变化过程，输出一个操作日志。介是目前action与reducer都不适合做这个逻辑操作，而react组件的功能也是主要用于显示的，也不适合。唯一可以做处理的就是在store插入中间个去处理。
我们的中间件相当于一个开箱即用的商品。即要就插入，不需要就拆除。操作简单，使用方便，不影响组件的正常功能。并且使用中间件还可以控制组件的变化，并且可以直接控制组件的变化与流程。
常用的中间件有redux-logger、redux-thunk、redux-promise、redux-saga等。
那需求中要求打印日志。则可以在createStore函数添加最后的参数做为中间件传入，接管createStore的功能，并返回一个store对象。
引入中间件及中间件包装函数applyMiddleware
```javascript
import { createStore, bindActionCreators, combineReducers, applyMiddleware } from 'redux';
import loggerMiddleware from 'redux-logger';
```
只要修改createStore函数则可：
```javascript
...
const store = createStore(combineReducers({counter, price}),applyMiddleware(loggerMiddleware));
...
```
关于中间件，也可以自己定义一个，感兴趣的同学可以看[自定义中间件](http://www.redux.org.cn/docs/api/applyMiddleware.html)。而关于中间件的应用，在此不详细介绍。

#### Chrome开发工具插件react、redux devtools
使用Chrome浏览作为我们的react应用的开发浏览器，可以安装react devtools与redux devtools，安装过程需要翻墙。有需要的同学可以去下载安装Chrome扩展。
安装完成了react devtools可以直接查看页面的结构变成了
```xml
<Provider store={...}>
    <Connenct(Counter)>...</Connenct(Counter)>
</Provider>
```
安装完成了redux devtools，则还需要在我们的代码里面配置一个全局属性。
需要引入复合函数compose进行将其与中间件拼合：
```javascript
...
const store = createStore(
    combineReducers({counter, price}),
    compose(
        applyMiddleware(loggerMiddleware),
        window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
    )
);
...
```






### immutable.js
对于react来说，引入了redux架构，已经解决了显示与数据分开处理问题。但是还有一个问题就是，当一个应用是由很多组件组成的。如果父组件数据更新了，但子件数据并没有更新。这时所以组件都重新渲染一次，这将是一项非常浪费性能的问题。如果我们能做到谁数据变化，就只更新谁，那就完美地解决了性能的问题。
毫无疑问，immutable.js将是我们最好的帮手。而immutable是何物？官网上称是：**不可以被改变的数据**，我的理解就是一旦创建，它将是不可以改变的。就好像定义一个恒量一样。比如es6中利用const定义的变量。利用其数据的不可变性，我们就可以实现做数据层面的state更新的时候，就不需要进行深拷贝，我们都知道深拷贝是非常耗内存的事（因为复杂对象的拷贝是要多层循环嵌套）。immutable的使用方式及实现原理可以去[官网](http://facebook.github.io/immutable-js/docs/)查看文档。
先安装immutable.js
```bash
$ yarn add immutable
```
这时我不在用对象的合成方式返回新的纯对象。而是返回一个immutable对象。整个数据层的流动，只传immutable对象，操作与修改都是不可变数据，将运行的效率达到极致。
```javascript
import React, { Component } from 'react';
import { render } from 'react-dom';
import { createStore, bindActionCreators, combineReducers } from 'redux';
import { Provider, connect } from 'react-redux';
import { Map } from 'immutable';

// Actions
const SUB = 'SUB';
const sub = () => ({type: SUB});
const ADD = 'ADD';
const add = () => ({type: ADD});

const PRICE = 'PRICE';
const inputPrice = price => ({type: PRICE, price});

// Reducers
const counter = (state = Map({counter: 0}), action) => {
    switch(action.type) {
        case SUB: return state.update('counter', v => v - 1);
        case ADD: return state.update('counter', v => v + 1);
        default: return state;
    }
}

const price = (state = Map({price: 1}), action) => {
    switch(action.type) {
        case PRICE: return state.update('price', () => action.price);
        default: return state;
    }
}

// store
const store = createStore(combineReducers({counter, price}));

// containers
@connect(
    state => state,
    dispatch => bindActionCreators({sub,add,inputPrice}, dispatch)
)
class Counter extends Component {
    render() {
        const { counter, price, sub, add, inputPrice } = this.props;
        const result = counter.get('counter') * price.get('price');
        return (
            <div>
                <div>
                    <input type="button" value="-" onClick={sub} />
                    <input type="text" value={counter.get('counter')} style={{width: "50px", textAlign: "center"}} />
                    <input type="button" value="+" onClick={add}/>
                </div>
                <input type="text" value={price.get('price')} style={{width: "94px", textAlign: "center"}} onChange={e => inputPrice(e.target.value)} />
                <p style={{width: "94px", textAlign: "center"}}>{result}</p>
            </div>
        )
    }
}

// render
render(
    <Provider store={store}>
        <Counter />
    </Provider>,
    document.getElementById('root')
)
```

### 异步Action数据流处理
上面所有的需求都是在不请求服务器的基础上做的，明显这种情况是很少的。实际开发中，基本上都与服务器请求有关，所以处理异步Action是不可避免的。
处理异步数据流，首先得定义异步的Action。上面的中间件中已经提到过applyMiddleware这个函数，此函数实际上就是将中间件做重新包装，最终在中间件里面实现同步的action发出。
#### 需求二
现有需求如下：
1. 设计一个手机号码属地查询功能；
2. 输入手机号，点查询，在下方显示电话归属地信息；
3. 手机归属地开放数据api接口：<http://tcc.taobao.com/cc/json/mobile_tel_segment.htm?tel=15850781443>，其中tel为传入的请求参数。

#### 异步Action
关于异步action的处理及原理，不多说了。下面说一下如何做这个需求。
根据要求先设计出同步Actions
```javascript
const GET_MOBILE = 'GET_MOBILE';
const getMobile = mobile => ({type: GET_MOBILE, mobile});

const FETCHING = 'FETCHING';
const fetchingMobile = fetching => ({type: FETCHING, fetching});
```
设计一个异步Action，用于请求接口数据，在此异步请求方式我们使用了fetch-jsonp插件
```javascript
$ yarn add fetch-jsonp
```
引入fetch-jsonp可以用于处理跨域请求数据（注意：跨域请求的数据必须要服务器提供jsonp格式的数据结构，否则会报token错误）。
```javascript
const fetchMobile = mobile => dispatch => {
    dispatch(fetchingMobile(true));
    return fetchJsonp(`https://tcc.taobao.com/cc/json/mobile_tel_segment.htm?tel=${mobile}`)
        .then(response => response.json())
        .then(json => json)
        .then(mobile => dispatch(getMobile(mobile)))
        .then(() => dispatch(fetchingMobile(false)))
}
```
#### 异步中间件
fetchMobile这个Action返回的结果实际上不在是一个单纯的js对象，而是一个函数。官方针对异步数据的处理提供了几个插件：redux-thunk、redux-promise、redux-saga，我们用比较简单的redux-thunk实现其功能。
安装redux-thunk插件：
```javascript
$ yarn add redux-thunk
```
利用此中间件包装我们的reducer，就可以实现处理异步action的方法。最终实现的逻辑为：
```javascript
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import { render } from 'react-dom';
import { createStore, combineReducers, bindActionCreators, applyMiddleware, compose } from 'redux';
import { Provider, connect } from 'react-redux';
import thunkMiddleware from 'redux-thunk';
import { Map } from 'immutable';
import fetchJsonp from 'fetch-jsonp';
import 'bootstrap/dist/css/bootstrap.css';

const GET_MOBILE = 'GET_MOBILE';
const getMobile = mobile => ({type: GET_MOBILE, mobile});

const FETCHING = 'FETCHING';
const fetchingMobile = fetching => ({type: FETCHING, fetching});


const fetchMobile = mobile => dispatch => {
    dispatch(fetchingMobile(true));
    return fetchJsonp(`https://tcc.taobao.com/cc/json/mobile_tel_segment.htm?tel=${mobile}`)
        .then(response => response.json())
        .then(json => json)
        .then(mobile => dispatch(getMobile(mobile)))
        .then(() => dispatch(fetchingMobile(false)))
}

const mobile = (state = Map({mobile: {}, fetching: false, list: []}), action) => {
    switch(action.type) {
        case GET_MOBILE: return state.update('mobile',() => action.mobile);
        case FETCHING: return state.update('fetching',() => action.fetching);
        default: return state;
    }
}


const store = createStore(combineReducers({mobile}), compose(applyMiddleware(thunkMiddleware), window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()));


const Search = ({ fetchMobile, isFetching }) => {
    let input;
    return (
        <div className="col-lg-12">
            <div className="input-group">
                <input type="text" className="form-control" placeholder="输入手机号码" ref={node => {input = node}}/>
                <span className="input-group-btn">
                    <button className="btn btn-default" type="button" onClick={() => fetchMobile(input.value)}>{ isFetching ? "查询中..." : "查询" }</button>
                </span>
            </div>
        </div>
    )
}
Search.propTypes = {
    fetchMobile: PropTypes.func.isRequired
}

const Infos = ({ mts, province, catName, telString, areaVid, ispVid, carrier }) => (
    <div className="col-lg-12" style={{marginTop: "30px"}}>
        <div className="jumbotron">
            <h1>{telString}</h1>
            <p>号码归属地：{province}</p>
            <p>运营商：{catName}</p>
        </div>
    </div>
)

@connect(state => state, dispatch => bindActionCreators({getMobile,fetchingMobile,fetchMobile},dispatch))
class App extends Component {
    render() {
        const { mobile, fetchMobile, fetchingMobile } = this.props;
        return (
            <div className="container" style={{marginTop: "20px"}}>
                <Search
                    fetchMobile={fetchMobile}
                    isFetching={mobile.get('fetching')}/>
                {!Map(mobile.get('mobile')).isEmpty() && <Infos {...mobile.get('mobile')} />}
            </div>
        )
    }
}

render(
    <Provider store={store}>
        <App />
    </Provider>,
    document.getElementById('root')
)
```


### 总结
关于react的全家桶，我在此省略了react-routor，因为我简略掉了页面结构。如果有兴趣的同学可以另外找资料看。
我对react+redux+immutable+webpack+css(less/sass)的介绍都比较简略，主要说的是如何实现我的需求，初学者可能有许多地方看得不是明了的，请移步[React学习资源汇总](https://github.com/tsrot/study-notes/blob/master/React%E5%AD%A6%E4%B9%A0%E8%B5%84%E6%BA%90%E6%B1%87%E6%80%BB.md)去看明白这些概念。学习的路还很长，我对react可以说算入了个门，还有很多概念及原理我也说不清楚，只知道这么用，我在些只是将我学会的东西都应用到我的demo中去，算是对自己有一个交待而已，也拿出来让大家吐槽一下。文中我用到了好多关于react家族成员或者沾边的知识点：react组件，无状态组件，redux架构，中间件，异步数据流，指令，不可变数据immutable的应用，复合函数等。说白了，就是要了解关于如何去学好函数式编程（FP），这些需要不断地练习才能慢慢从代码中体会。