---
title: 【从一道面试题说起】函数节流
categories:
  - 一道面试题
date: 2018-09-14 10:26:18
tags:
---

某些情况下，函数调用并非是用户自身能控制的，可能会造成性能浪费。

## 使用场景

### window.onresize

监听window的resize事件时，随着浏览器窗口的大小改变，触发的频率非常高。如果在window.resize函数中进行了dom操作，这会对性能造成非常大的浪费，会造成浏览器卡顿。

### mousemove

给一个div绑定mousemove事件，那么在鼠标滑动的时候，会频繁触动拖曳事件。

## 函数节流原理

设置每多少秒才触发一次事件。比如window.resize事件，在1s内会触发10次，实际上我们只需要2-3次，所以我们可以设置每500ms才触发一次。可以借助setTimeout来实现。

## 代码实现

把下面的代码放浏览器console中，并拖动浏览器窗口。

``` javascript
var throttle = function(fn, interval) {
    var timer = null;
    var self = fn;
    var isFirstTime = true;
    return function() {
        var me = this;
        // 第一次调用，不用节流
        if (isFirstTime) {
            self.apply(me, arguments);
            return isFirstTime = false;
        }
        // 如果上次计时器还在，节流
        if (timer) {
            return false;
        } 
        
        timer = setInterval(function() {
            clearTimeout(timer);
            self.apply(me, arguments);
            timer = null;
        }, interval); 
    }
}

window.onresize = throttle(function() {
    console.log(1);
}, 500);
```
