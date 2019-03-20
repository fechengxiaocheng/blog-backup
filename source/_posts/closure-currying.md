---
title: 【从一道面试题说起】闭包&函数柯里化
categories:
  - 一道面试题
date: 2018-09-11 12:06:00
tags:
---


## 题目

### 考察内容

闭包，函数柯里化

### 题目描述

* 写一个func函数，调用func(-, b, c)(a)，按照顺序输出a,b,c
* 写一个func函数，调用func(-, b, c)(a)()，按照顺序输出a,b,c

## 代码

``` javascript
// func(-, b, c)(a)，按照顺序输出a,b,c
function f() {
    let args = [].slice.call(arguments);
    let str = args.join('');
    const i = str.indexOf('-');
    if (i >= 0) {
        return function(x) {
            args.splice(i,1,x);
            return args.join('');
        }
    }
}
console.log(f(1,'-', 3)(2))
```

<!--more-->

```javascript
// 调用func(-, b, c)(a)()，按照顺序输出a,b,c
var f = (function() {
    let args = [];
    let order = -1;
    const _f = function() {
        let flag = false; // 是否有下划线
        for (let i = 0 ;i < arguments.length; i++) {
            if (arguments[i] === '-') {
                flag = true;
                order = i;
            }
        }
        console.log('order...', order, flag)
        if (flag) {
            args = [].concat.call(args, [].slice.call(arguments));
            return _f;
        } else {
            // 没有下划线了，当前的arguments替换order处的下划线，返回值
            const current = [].slice.call(arguments);
            [].splice.call(args, order, 1, current[0]);
            return args.join('');
        }
    }
    return _f;
})()

console.log(f(1,'-', 3)(2))

```

## 函数柯里化

### 介绍

currying的概念最早是由俄国数学家发明，后由著名的数理逻辑家Haskell Curry将其丰富发展，curring由次得名。currying又称部分求值。一个currying接收到一批参数之后，并不直接求值，而是继续返回另一个函数，之前的参数通过闭包保存在内存中。直到最后一次调用的时候，返回所有参数的求值。

### 使用场景

使用场景，小A每天都要记录存款，add(100), add(120)...直到一个月最后一天的时候再去计算总存款数add()。不用每天都算一遍。

### 代码


小A 到最后arguments.length === 0的时候，才返回总的存款。之前都把每天花了多少钱记录在内存中。

``` javascript
var add = (function() {
    var args = [];
    return function() {
        if (arguments.length === 0) {
            var sum = 0;
            for(var i = 0; i < args.length; i++) {
                sum += args[i];
            }
            return sum;
        } else {
            [].push.call(args, arguments[0]);
            return arguments.callee;
        }
    }
})();
```

下面思考下，如果把柯里化函数做到更通用，很容易想到可以把sum相加部分的代码单独抽离封装起来。
然后通过入参fn传入柯里化函数。

``` javascript
var currying = function(fn) {
    var args = [];
    return function() {
        if (arguments.length === 0) {
            return fn.apply(this, args);
        } else {
            [].push.apply(args, arguments);
            return arguments.callee;
        }
    }
};

var add = (function() {
    var sum = 0;
    return function() {
        for(var i = 0; i < arguments.length; i++) {
            sum += arguments[i];
        }
        return sum;
    }
})();

var add = currying(add);

console.log(add(1)(2)(3)())

```

## 闭包和函数柯里化到底有何关系

* 函数柯里化是积累参数，延迟到最后执行add操作; 

* 函数柯里化也是通过闭包概念实现的，积累的参数可以保存在内存中。


## 练习巩固

### 题目一

* 题目1、add(1)(2)(3)(4); // 10
* 题目2、add(1)(2)(3)(4)(); //10
* 题目3、add(1,2)(3,4)(); //10
* 题目4、add(1,2)(3,4)(5,6).toString(); //10

#### 闭包实现

``` javascript
/**
 * @description 闭包实现题目1
 * @param {number} num 
 * @returns {object}
 */
function add(x) {
    return function(y) {
        return function(z) {
            return function(w) {
                return x + y + z + w;
            }
        }
    }
}

console.log(add(1)(2)(3)(4));
```

``` javascript
/**
 * @description 闭包实现题目2
 * @param {number} x 
 * @returns {function}
 */
function add(x) {
    var sum = 0;
    sum += x;
    return function(y) {
        if (arguments.length === 0) {
            return sum;
        } else {
            sum += y;
            return arguments.callee;
        }

    }
}

console.log(add(1)(2)(3)(4)());
```


```javascript

/**
 * @description 闭包实现题目3
 * @returns {function}
 */
function add() {
    var sum = 0;
    var args = [].slice.call(arguments);
    sum += args.reduce(function(a,b) {return a+b});
    return function() {
        if (arguments.length === 0) {
            return sum;
        } else {
            var _args = [].slice.call(arguments);
            sum += _args.reduce(function(a,b) {return a+b});
            return arguments.callee;
        }
    }
}

console.log(add(1,2)(3,4)());

```
``` javascript

/**
 * @description 闭包实现add(1,2)(3,4)(5,6).toString()
 * @returns {function}
 */
function add() {
    var sum = 0;
    var args = [].slice.call(arguments);
    sum += args.reduce(function(a,b) {return a+b});
    var _add = function() {
        var _args = [].slice.call(arguments);
        sum += _args.reduce(function(a,b) {return a+b});
        return _add;
    }
    _add.toString = function() {
        return sum;
    }
    return _add
}

console.log(add(1,2)(3,4)(5,6).toString());

```

#### 函数柯里化实现

```javascript
/**
 * @description 函数柯里化实现题目2
 * @param {number} x 
 * @returns {function}
 */
function add() {
    var args = Array.prototype.slice.call(arguments);
    var _add  = function () {
        var _args =  Array.prototype.slice.call(arguments);
        if (arguments.length === 0) {
            return args.reduce(function(a,b) {return a+b;});
        } else {
            Array.prototype.push.call(args, _args[0]);
            return _add;
        }
    };
    return _add;
    
}

console.log(add(1)(2)(3)(4)());
```


``` javascript
/**
 * @description 函数柯里化实现题目3
 * @returns {function}
 */
function add() {
    var args = Array.prototype.slice.call(arguments);
    var _add = function() {
        var _args = Array.prototype.slice.call(arguments);
        if (arguments.length === 0) {
            return args.reduce(function(a,b) {return a + b});
        } else {
            args = Array.prototype.concat.call(args, _args);
            return _add;
        }
    };
    return _add;
    
}

console.log(add(1,2)(3,4)());
```

### 题目二

思考以下两块代码分别输出什么

``` javascript
var name = "The Window";

var object = {
    name : "My Object",

    getNameFunc : function(){
        return function(){
            return this;
　　　　　};
　　　}
};

console.log(object.getNameFunc()());
```

``` javascript
var name = "The Window";

var object = {
    name : "My Object",

    getNameFunc : function(){
        var that = this;
        return function(){
            return that.name;
　　　　　};
　　　}
};

console.log(object.getNameFunc()());
```

### 题目三

思考以下代码输出什么：

``` javascript
function buildList(list) {
    var result = [];
    for (var i = 0; i < list.length; i++) {
        var item = 'item' + list[i];
        console.log(item, list[i]);
        result.push( function() {console.log(item + ' ' + list[i])} );
    }
    console.log(result);
    return result;
}
 
function testList() {
    var fnlist = buildList([1,2,3]);
    for (var j = 0; j < fnlist.length; j++) {
        fnlist[j]();
    }
}

testList();
```

## 参考文档

[阮一峰学习Javascript闭包（Closure）](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)
[理解javascript闭包](https://coolshell.cn/articles/6731.html)
[一道javascript面试题（闭包与函数柯里化）](https://www.cnblogs.com/qcblog/p/6858947.html)



