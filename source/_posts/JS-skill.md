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

###利用按位~简写条件
普及一下`~`运算符，这个符号学名叫“按位非”，它是一个一元运算符。按位非操作符由一个波浪线（~）表示，执行按位非的结果就是返回数值的反码，结果总是一个*数字*。按位非运算符，简单的理解就是改变运算数的符号并减去1。
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
> Tips: 先声明一下，此技巧我不太推荐，如果想装下逼，就用用，但可能会被喷死。毕竟，这种方式不太常用，让读代码的人感觉到困惑。

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

### 利用!!强转布尔类型
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

### 利用&&当条件
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
```

### 利用||取值
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
如果直接使用`||`或运算符，则可以直接取值。
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

### |

### +

### !

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

## 方法的妙用

### isNaN

