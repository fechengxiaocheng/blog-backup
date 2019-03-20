---
title: 【解读源码系列】Function.prototype.bind
categories:
  - 源码系列
date: 2018-09-12 10:36:48
tags:
---

bind用来绑定函数内部的this指向。在某些浏览器不支持的情况下，需要polyfill自行去实现。

## 思路

先把调用bind的函数保存起来，在调用func()的时候执行bind中的闭包函数，返回func函数调用后return的值。

## 代码实现

``` javascript
Function.prototype.bind = function(context) {
    var self = this;
    return function() {
        return self.call(context, arguments);
    }
}
var obj = {
     name: 'xlj'
};

var func = function() {
    return this.name;
}.bind(obj);

console.log(func());
```

下面实现的难一点，在bind和func中都传入参数，bind(obj, 1, 2)，func(3,4)，要求输出[1,2,3,4].

``` javascript
Function.prototype.bind = function() {
    var self = this;
    var context = [].shift.call(arguments);
    var args = [].slice.call(arguments);
    return function() {
        return self.call(context, [].concat.call(args, [].slice.call(arguments)));
    }
}
var obj = {
     name: 'xlj'
};

var func = function() {
    console.log(this.name);
    console.log(arguments[0]);
}.bind(obj,1,2);

func(3,4);
```


