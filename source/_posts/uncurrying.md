---
title: 函数反柯里化uncurrying
categories:
  - 学习笔记
date: 2018-09-14 10:41:17
tags:
comments: true
---

我们在[【从一道面试题说起】闭包&函数柯里化】](http://goddess.joeray61.com/2018/09/11/closure-currying/)认识了函数科里化，本文我们来了解下函数反柯里化。

函数反柯里化其实就是把this泛化的过程。

在认识它之前，首先思考一个问题，如何让类数组借用数组的push方法，应该很容易想到,可以借助Array.prototype.push方法。

``` javascript
(function() {
    Array.prototype.push.call(arguments, 4);
    console.log(Array.prototype.slice.call(arguments)); // [ 1, 2, 3, 4 ]
})(1,2,3);
```

<!--more-->

继续思考，如何实现一个push函数，使得调用push([1,2,3], 4)之后，a数组变为[1, 2, 3, 4]。

``` javascript
const a = [1,2,3]
push(a, 4);
console.log(a); // [1, 2, 3, 4]
```

其实要做的无非就是封装一个方法，返回一个函数赋值给push，这个函数接收两个参数,arr和value，执行Array.prototype.push，把第一个参数arr作为this对象，value作为要push的值。

``` javascript
function uncurrying() {
    return function(arr, value) {
        return Array.prototype.push.call(arr, value);
    }
}
const push = uncurrying();
push(a, 4);
console.log(a); // [1, 2, 3, 4]
```

再思考，这样封装真的达到可变和不变的分离了么？并没有。如果再要实现一个slice(a,1)呢？那不得再写一个新的uncurrying函数？所以，对于uncurrying函数至少还有两点可以改进：

* 返回的闭包函数中Array.prototype.push抽离到uncurrying函数外面
* 闭包函数中的不用实参，arguments[0]为指定的this值，此外都是传入方法中的实际参数。

这样一来，方法中的this，不仅仅局限于定义时候的对象，而是加以泛化。这个就是函数反柯里化方法。也就是说uncurrying就是 **把this泛化的过程提取出来** 的方法。

``` javascript
const a = [1,2,3];
// 把uncurrying作为Function.prototype的一个原型方法，供Array.prototype.push等方法调用。
Function.prototype.uncurrying = function() {
    const self = this; // 把Array.prototype.push保存起来
    return function(arr, value) {
        const obj = Array.prototype.shift.call(arguments, 1);
        // 这里才是真正调用Array.prototype.push方法
        return self.call(obj, arguments[0]);
    }
}

var push = Array.prototype.push.uncurrying();
var slice = Array.prototype.slice.uncurrying();
var includes = Array.prototype.includes.uncurrying();

(function() {
    push(arguments, 4);
    console.log('a', Array.prototype.slice.call(arguments));
})(1,2,3);
console.log('b', slice(a, 1));
console.log('c', includes(a, 2));
// a [ 1, 2, 3, 4 ]
// b [ 2, 3 ]
// c true
```






