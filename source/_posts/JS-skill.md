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

### 利用~~取整

如果要将一个小数取整数部分，正常的逻辑是：`parseInt()`强转为整数方法，或者`Math.floor()`向下取整。其实，使用`~~`操作符可以更快速的取整。
```javascript
parseInt(12.55) //12

Math.floor(12.55) //12

~~12.55 //12
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
var a = (b === 0 && 1) || (b === 1 && 2) || 3
```

### |

### +

### !

## 原生方法

### isNaN

### typeOf