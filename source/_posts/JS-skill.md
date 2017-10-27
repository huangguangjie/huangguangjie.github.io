---
title: JS装逼实用技巧
categories:
- 技术
tags:
- javascript
- 总结
date: 2017/10/25
foldmethod: syntax
---

评价一段代码的好坏，往往是看代码的几个点：1、简洁易读，优雅的代码，读起来赏心悦目，更易于让人理解。2、执行效率，高效的执行速度，让代码的性能更省时间，省内存。3、注释，每个人的思维模式都不同，如果适当加一些注释，可以使你的代码及编程思维，更容易让人接受。
通过一些工作积累，我在项目开发中，也意识到写代码，不是做完就了事，还要重新阅读自己写的代码，并且考虑是否可以进行优化，这样，才是健壮的代码。如果普通的一些用法及语法，这里不再多说，现只把一些工作中逼格比较高的实用经验分享出来，感兴趣的同学，可以一观。让我们来见识到JS强大的表现力吧！

## 操作符的妙用

### 利用（按位与）&判断奇偶数
按位与运算的逻辑是这样： 0001 & 0011 = 0001，也就是两个位都是1，才是1，其它位都是0。
我们经常要做一个条件，判断一个数的奇偶性，会这样写：
```javascript
function fn(n) {
    if(n % 2 === 1) {
        //奇数
    } else {
        //偶数
    }
}
```
利用按位与运算，则可以简写为这样：
```javascript
function fn(n) {
    if(n & 1) {
        //奇数
    } else {
        //偶数
    }
}
```

### 利用（按位或）|取整
按位或运算的逻辑是这样： 0001 | 0011 = 0011，也就是两个位都是0，才是0，其它位都1。
```javascript
0 | 0 //0
0 | 1 //1
1 | 0 //1
1 | 1 //1
3 | 5 //7 即 0000 0011 | 0000 0101 = 0000 0111   因此，3|5的值得7
```
`|`有一个作用，可以用来数字取整。
```javascript
1.1 | 0 //1
-2.0 | 0 //-2
null | 0 //0
true | 0 //1
```
> Tips: 其实浮点数是不支持位运算的，所以会先把1.1转成整数1再进行位运算，就好像是对浮点数向下求整。所以1 | 0的结果就是1。

还有一个就是在设计vue组件的时候，也常用到的地方如下：（具体为啥这么用，我也说不清楚，没研究过原理）。
```javascript
export default {
    props {
        params: Object | Array //只要其中任意一个满足条件即可
    }
}
```

### 利用(按位非)~简化表达式
普及一下`~`运算符，这个符号学名叫“按位非”，它是一个一元运算符。按位非操作符由一个波浪线（~）表示，按位非就是求二进制的反码。不管什么值使用了这个运算符，结果总是一个*数字*。按位非运算符，简单的理解就是改变运算数的符号并减去1。
```javascript
~5 //-6
~0 //-1
~-2 //1
~true //-2 这里true被强转为1
~null //-1 这里null被强转为0
~undefined //-1 这里undefined被强转为0
```
利用这个原理，我们可以在实际工作中这样应用：
```javascript
var str = 'hello world!'
if(str.indexOf('w') != -1) { /*...*/ }
//或者
if(str.indexOf('w') > -1) { /*...*/ }

//根据str.indexOf('w')的值，无外乎两种情况： -1, 0及正整数，
//则，从-1到正整数中，经过按位运算~之后，则为：0或者任意负数，
//实际表示为：false与true
//上面的条件语句，可以改为：
if(~str.indexOf('w')) { /*...*/ }
```
> Tips: 按位运算，简单的地方，可以使用。但一些比较复杂的、难理解的，我觉得应该尽量少用，因为会给阅读者带来困难，也会给自己带来麻烦。

### 左移<<求2的n次方
```javascript
//二进制运算：
01 << 2 //0100，十进制为4

//实际运用可以这样：
1 << 2 //4 即2的平方
1 << 3 //8 即2的立方
1 << 4 //16 即2的4次方
```

### 无符号右移>>>判断数的正负
正数的无符号右移与有符号右移结果是一样的。负数的无符号右移会把符号位也一起移动，而且无符号右移会把负数的二进制码当成正数的二进制码。即：
```javascript
1 >>> 0 //1
2 >>> 0 //2
4 >>> 0 //4
4 >>> 1 //2
4 >>> 2 //1
-4 >>> 0 //4294967292
-4 >>> 1 //2147483646
-2 >>> 0 //4294967294
-1 >>> 0 //4294967295
```
观察上面的例子，我们得出一个结论，正数右移0位，值不变，而负数右移0位，值已经变化了。即可以通过这种关系判断一个数的正负：
```javascript
function isPos(n) {
    return n === n >>> 0
}
isPos(-1) //false
isPos(1) //true
```


### 利用&&连结条件与表达式
通常，我们为了给一个变量赋值，正常的逻辑是：`if`语句判断，通过if逻辑控制语句，则赋值成功。利用`&&`符号，则可以直接在前面写上条件语句，后面写上执行语句。
```javascript
var a;
if(true) {
    a = 10
}

//也可以写成
var a;
true && (a = 10);
//或者
var a = true && 10; //这种情况尽量不要使用，因为如果条件不成立，则会给a赋上一个false值。

//常见使用：
function fn(obj) {
    obj && (return true);
    return false;
}

var a = b = c = true, d;
a && b && c && (d = 10);
console.log(d) //10
```
> Tips: 对于&&，需要注意以下几点：
1. 对于布尔值，逻辑与是非常简单的，只要有一个false，就返回false；
2. 对于不是布尔值的情况则：
    如果第一个操作数是对象，则返回第二个数
    如果第二个操作数是对象，则只有在第一个操作数的求值结果为true的情况下才会返回该对象；
    如果第两个操作数都是对象，则返回第二个数操作数
    如果有一个操作数是null，则返回null
    如果有一个操作数是NaN，则返回第NaN
    如果第一个操作数是undefined，则返回undefined

### 利用||取值（设置默认值）
通常，我们声明一个变量，可能要根据条件进行赋值。正常的逻辑是：`if ... else if ... else`或者使用`switch`这两种语句块。
```javascript
var a, b;
if(b === 0) {
    a = 1
} else if(b === 1) {
    a = 2
} else {
    a = 3
}
//或者
var a, b = 1;
switch(b) {
    case 0:
        a = 1;
        break;
    case 1:
        a = 2;
        break;
    default:
        a = 3;
        break;
}
```
如果直接使用`||`或运算符，则按从左到右，取第一个非空的一个值，否则直接使用false作为值使用。
```javascript
var b = 4;
var a = (b === 0 && 1) || (b === 1 && 2) || 3; // 3

//常用到的或运算取值方式一般是在函数里面。
function doSomething(obj) {
    var o = obj || {} //如果obj不存在，则使用默认值
    var oo;
    o && (oo = obj || {}) //结合与运算，作变判断的前置条件
}
```
> Tips: 关于||，需要注意以下几点：
1. 对于布尔值，逻辑或是非常简单的，只要有一个true，就返回true；
2. 对于不是布尔值的情况则：
    如果第一个操作数是对象，则返第一个操作数
    如果第一个操作数的求值结果为false，则返回第二个操作数
    如果两个操作数都是对象，则返回第一个操作数
    如果两个操作数是null，则返回null
    如果两个操作数是NaN，则返回NaN
    如果两个操作数是undefined，则返回undefined 

### 利用~~取整
如果要将一个小数取整数部分，正常的逻辑是：`parseInt()`强转为整数方法，或者`Math.floor()`向下取整。其实，使用`~~`操作符可以更快速的取整。
```javascript
parseInt(12.55) //12
Math.floor(12.55) //12
~~12.55 //12
```
其实，除了取整的作用，它还可以达到强转数字的作用，比如：
```javascript
~~true //1
~~false //0
~~[] //0
~~{} //0
~~undefined //0
~~!undefined //1
~~null //0
~~!null //1
```
> Tips: 虽然~~用起来比较骚气，但是为了可读性，本人还是建议使用Math.floor()更为稳妥，谁知道领导review代码的时候，会不会说你过于装逼，要被喷死。

### 利用+将字符串转为数字
如果要将一个表现字符串的数字转化为真正的数字，正常逻辑是：`Number()`或者`parseInt()、parseFloat()`实现转化。实际上，我们还可以更简捷，只需要在前面添加一个`+`就可以了。当然，你也可以用`-`来实现，只不过，这样子，则值就成了负数。
```javascript
Number('123') //123
parseInt('123') //123
parseFloat('123') //123
parseFloat('123.0') //123
parseFloat('123.0') //123
parseFloat('123.1') //123.1

//只要使用+，则可以实现数字的转化
+'123' //123
+'12.22' //12
-'12.1' //-12.1

//如果要取整并且转数字的话，使用~~
~~'123.33' //123
```
实际上，`+`也适用于Date对象。
```javascript
var date = new Date //Fri Oct 27 2017 14:38:49 GMT+0800 (中国标准时间)
date.getTime() //1509086332914

//使用+直接输出时间缀
+new Date //1509086332914
```

### 利用!逻辑非强转布尔值
逻辑非会将所有值，转为布尔值：
```javascript
!{} //false 一个空对象，实际上是一个引用，属于存在的引用地址值
![] //false 存在引用地址值
!'' //true
!0 //true
!'hello' //false
!null //true
!NaN //true
!undefined //true
```
利用逻辑非运算符，可以省去一些多余的判断。比如经常要判断一个值非空：
```javascript
var a, b;
if(a !== undefined && a !== null && a !== '' && a !== {} && a!== 0 && a!== undefined) {
    b = true;
}
```
实际上，如果像这面这种判断，只需要一个逻辑非。
```javascript
var a, b;
!a && (b = true);
```


### 利用!!实现变量检测
如果要将一个值强转为布尔类型，正常的逻辑是：`Boolean()`强转为布尔值。
```javascript
Boolean(123) //true
Boolean('hello') //true
Boolean('false') //true
Boolean(null) //false
Boolean(undefined) //false
Boolean('undefined') //true

```
然而，通过`!!`两个非逻辑符，则可以将一个值强转为布尔值。
```javascript
!!123 //true
!!'hello' //true
!!'false' //true
!!null //false
!!undefined //false
!!0 //false 注意，0也会转为false，在数字中，只有0会转为false，其它非0值，都会转为true
!!'' //false
!!NaN //false
```
> Tips: 任意的javascript的值都可以转换成布尔值。这些值会被转换成false：undefined,null,0,-0,NaN,""，而其它都变强转为true。通常，利用`!!`符号来检测一个变量是否存在。

## 表达式的妙用
### 自执行函数
函数，只是声明，并不能直接执行，需要调用才会执行。而如果，变成了表达式，则会自动执行。自执行函数，则是利用了表达式的这种特性。那些匿名函数附近使用括号或一些一元运算符来引导解析器，指明运算符附近是一个表达式。
按照这个理解，可以举出五类，超过十几种的让匿名函数表达式立即调用的写法：
```javascript
( function() {}() );
( function() {} )();
[ function() {}() ];

~ function() {}();
! function() {}();
+ function() {}();
- function() {}();

delete function() {}();
typeof function() {}();
void function() {}();
new function() {}();
new function() {};

var f = function() {}();

1, function() {}();
1 ^ function() {}();
1 > function() {}();
```
> Tips：另外值得再次注意的是，括号的含混使用——它可以用来执行一个函数，还可以做为分组运算符来对表达式求值。比如使用圆括号或方括号的话，可以在行首加一个分号，避免被用做函数执行或下标运算：
```javascript
;( function() {}() )
;(function(){})()
```

### isNaN判断是否为合法数字
isNaN(x) 函数用于检查其参数是否是非数字值。如果`x`是特殊的非数字值NaN（或者能被转换为这样的值），返回的值就是 true。如果`x`是其他值,则返回 false。
经常，我们在页面功能模块开发过程中，拿到的交互数据，有很大可能是字符串形式的“数字”。这时，我们如果要做计算，就得先判断是否合法的数字。
```javascript
isNaN('111') //false
isNaN(111) //false
isNaN(12.2) //false
isNaN('12.2') //false
isNaN('aa11') //true
isNaN(undefined) //true 一切非数字，返回都是true
```
> Tips: 我们可以这样总结，只要是表现得像数字（字符串形式，或者真正数字）的都可以检测出是属于“数字”,否则属于不合法数字。

### 缓存Array.length
写for循环的时候，经常是这样：
```javascript
var arr = [1,2,3]
for(var i = 0; i < arr.length; i++) {/**/}
```
在处理一个很大的数组循环时，对性能的影响将是非常大的。为了提升运行性能，需要将数组使用一个变量缓存起来使用。
```javascript
for(var i = 0, len = arr.length; i < len; i++){/**/}
```

### 判断属于是否存在于对象中
1、使用in运算符
```javascript
var obj = { a: 11 }
'a' in obj //true
'b' in obj //false
```
2、使用!!
```javascript
var obj = { a: 11 }
!!obj.a //true
!!obj.b //false

//或者使用undefined来判断，但是可能也会属于值本身就是undefined，这样子就判断不出来了。
obj.a !== undefined //true
obj.b !== undefined //false
```
3、hasOwnProperty()方法
```javascript
var obj = { a: 11 }
obj.hasOwnProperty('a') //true
obj.hasOwnProperty('b') //false
//该方法只能判断自有属性是否存在，对于继承属性会返回false。
```

### 获取数组最后一个元素
获取数组最后一个元素的方式有多种，常用的有`array.pop`或者`array[array.length - 1]`方式。实际上，还可使用`Array.slice(-1)`
```javascript
var arr = [1,2,3,4,5]
arr.pop() //5 此方法会改变数组的结构，不推荐
arr[arr.length - 1] //5
arr.slice(-1)[0] //5 不需要计算数组的长度，直接拿到最后一个元素
```

### 数组截断
数组有时候需要设置一个上限，或者删除数组中的一些元素，使用array.length = [长度值] 这种方式非常有用。
```javascript
var arr = [1,2,3,4,5]
arr.slice(0,3)
console.log(arr) //[1,2,3]
arr = arr.splice(0,3) //splice方法会改变数组结构
console.log(arr) //[1,2,3]

//直接设置长度值
arr.length = 3
console.log(arr) //[1,2,3]

//还可以实现数组的清空操作
arr.length = 0; //arr => []
```

### 数组合并
合并数组常用的方法是：`concat`。不过现有一种更快速的合并方式：`Array.push.apply(arr1,arr2)`
```javascript
var arr1 = [1,2,3], arr2 = [4,5];

arr1 = arr1.concat(arr2) //常规方式
//或者
arr1 = arr1.push.apply(arr1,arr2); //装逼方式，但运行速度更快！

console.log(arr1) //[1,2,3,4,5] 两种方式都可以达到合并的结果
```

### 类数组利用数组的方法
类数组拥有length属性，但不具有数组所具有的方法。为了方便操作，常需要将其转化为数组结构。
```javascript
var arrLike = { '0': 'a', '1': 'b', length: 2 }; //下标为 '0' '1'才符合数组的数据结构
Array.prototype.join.call(arrLike, '-'); //a-b 模拟数组的join()方法,使用-分隔。
Array.prototype.slice.call(arrLike); //['a','b'] //模拟数组slice方法，返回一个新的数组

function fn() {
    var args = Array.prototype.slice.call(arguments); //获取所有参数的列表
}
```

### 利用arguments.callee实现递归
普通的递归方法是这样写的：
```javascript
function fn(n) {
    if(n <= 1) {
        return 1
    } else {
        return n * fn(n - 1)
    }
}
```
但是当这个函数变成了一个匿名函数时，我们就可以利用callee来递归这个函数。
```javascript
function fn(n) {
    if(n <= 1) {
        return 1
    } else {
        return n * arguments.callee(n - 1)
    }
}
```
> Tips: 这个方法虽然好用，但是有一点值得注意，ECMAScript4中为了限制js的灵活度，让js变得严格，新增了严格模式，在严格模式中我们被禁止不使用var来直接声明一个全局变量，当然这不是重点，重点是arguments.callee这个属性也被禁止了。

### 给回调函数传递参数
经常，函数一般是可以当作参数来使用。但是有时候，一些函数自身需要带参，这时候，把函数当参数使用的话，就显得比较麻烦。一般的处理方法是，通过附加的传参的办法。
```javascript
function callback(obj) {
    console.log(obj)
}

function fn(callback, obj) {
    callback(obj)
}
```
但是有些时候，函数的参数限定了，只能传一个回调函数，这时，这种附加传参的显得无力了。采用闭包的方式可以解决此问题。
```javascript
function callback(obj) {
    return function() {
        console.log(obj)
    }
}

document.body.addEventListener('click',callback('hello')); //执行callback('hello')，则返回的是一个函数
```

