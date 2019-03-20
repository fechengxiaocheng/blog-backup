---
title: 【从一道面试题说起】用原生js实现AOP（面向切面编程）
categories:
  - 一道面试题
date: 2018-09-12 10:39:04
tags:
---

## 题目

### 考查内容

在具体项目中，经常会碰到需要把主业务逻辑和无关功能分开(无关业务逻辑的如日志、处理异常)，这个叫做AOP（面向切面编程)。通过把一个函数动态织入另一个函数来实现。这样的好处是：1、提高了业务代码的纯度，2、非业务代码的高度复用和维护。这种AOP的方式给函数添加职责，也是一种巧妙的装饰者模式实现。

### 题目描述

实现一个func函数：

* 主业务逻辑 console.log(2); 
* 与业务无关逻辑：在主业务逻辑之前console.log(1)；在主业务逻辑之后输出console.log(3)。

在调用func()的时候依次输出1，2，3。


## 思路

扩展Function的方法：before和after，before中先执行bofore中的与业务无关逻辑，再执行主业务逻辑；after相反，先执行当前主业务逻辑，在执行与业务无关逻辑。

## 代码实现

``` javascript
Function.prototype.before = function(fn) {
    var self = this; // 保存当前的func
    return function() {
        fn.apply(this, arguments); // 执行before中的入参函数
        return self.apply(this, arguments) // 执行并返回当前函数
    }
}

Function.prototype.after = function(fn) {
    var self = this; // 保存当前的func
    return function() {
        var ret = self.apply(this, arguments) // 执行当前函数
        fn.apply(this, arguments); // 执行after中的入参函数
        return ret;
    }
}

var func = function() {
    console.log(2);
};

func = func.before(function() {
    console.log(1);
}).after(function() {
    console.log(3);
});

func();
```


