---
title: 【解读源码系列】Array.from
categories:
  - 源码系列
date: 2018-09-20 14:37:48
tags:
---

Array.from可通过1、类数组(arguments)2、可迭代对象(set,map)来创建新数组。

``` javascript
// 后面两个参数可选
Array.from(arrayLike, mapFn, thisArg) 
```

* arrayLike: 必选，1、类数组(argumentg)2、可迭代对象(set,map)。
* mapFn: 可选，相当于Array.from(arrayLike).map(mapFn, thisArg)。
* thisArg: 可选，执行回调函数mapFn时候的this对象。非常有用，利于解耦。可以把被处理的数据和对象分离，thisArg中定义处理函数handle，用来在mapFn中返回调用handle之后的结果。

<!--more-->

## 用法

### string
 
``` javascript
const a = Array.from('abc');
console.log(a); // [ 'a', 'b', 'c' ]
```

### new Set()

``` javascript
const b = Array.from(new Set([1,2,3]));
console.log(b); // [ 1, 2, 3 ]
```

### new Map()

``` javascript
const c = Array.from(new Map([[1, 2], [3, 4]]));
console.log(c); // [ [ 1, 2 ], [ 3, 4 ] ]
```

### 类数组

``` javascript
const d = (function() {
    const d1 = Array.from(arguments);
    console.log(d1);
})(1,2,3); // [ 1, 2, 3 ]
```

### 使用第二个参数func返回新函数

``` javascript
const e = Array.from([1,2,3], x => x*2);
console.log(e); // [ 2, 4, 6 ]
```

``` javascript
const f = Array.from({length: 5}, (v, i) => i + 1);
console.log(f); // [ 1, 2, 3, 4, 5 ]
```

### 数组去重合并

``` javascript
const g1 = [1,2,3,4];
const g2 = [3,4,5,6];
const g3 = [5,6,7,8];
const g = (function() {
    const _g = [].concat.apply([], arguments);
    console.log(Array.from(new Set(_g)));
})(g1, g2, g3); // [ 1, 2, 3, 4, 5, 6, 7, 8 ]
```

### 充分利用第三个参数thisArg

``` javascript
const obj = {
    handle: x => x * 3
}
console.log(Array.from([1,2,3], function(x){ return this.handle(x) }, obj))
// [3, 6, 9]
```

## 思路

* 判断如果第一个参数arrayLike为空，throw new TypeError("Array.from requires an array-like object - not null or undefined");

* 循环arrayLike，判断如果有第二个参数mapFn，则返回arr[i] = mapFn(arrayLike[i], i); 否则直接返回arrayLike[i]

* 判断如果有第三个参数，mapFn调用的时候mapFn.apply(this, arrayLike[i], i)

* 返回arr

ps: polyfill的代码和底层实现的未必是完全一致的。比如这里使用new Set[1,2,3]和new Map[[a,1],[b,1]]返回的值都是[]。因为能支持es6代码的浏览器基本都不需要polyfill实现Array.from。

## 代码实现

``` javascript
Array.from = (function() {
    const isCallable = function(fn) {
        return typeof fn === 'function' && Object.prototype.toString.call(fn) === '[object Function]';
    };
    // 返回一个value的整数
    const toInteger = function(value) {
        const v = Number(value);
        if (isNaN(v)) {
            return 0;
        }
        // 0或者无穷大的数，直接返回
        if (v === 0 || !isFinite(v)) {
            return v;
        }
        return ( v > 0 ? 1 : -1 ) * Math.floor(Math.abs(v));
    }
    const maxSafeInteger = Math.pow(2, 53) - 1;
    const toLength = function(value) {
        const len = toInteger(value);
        // len的最小值不能比0小。最大值不能比maxSafeInteger大。
        return Math.min(Math.max(len, 0), maxSafeInteger);
    }
    return function (arrayLike/*, mapFn, thisArg*/) {
        const C = this;
        // 如果没有第一个参数，throw error。
        if (arrayLike == null) {
            throw new TypeError("Array.from requires an array-like object - not null or undefined");
        }
        const items = Object(arrayLike);
        let thisArg = '';
        const mapFn = arguments.length > 1 ? arguments[1] : void undefined; 
        if (typeof mapFn !== 'undefined') {
            // 如果有第二个参数，判断是第二个参数类型如果不是构造函数，throw error
            if (!isCallable(mapFn)) {
                throw new TypeError("Array.from when provided mapFn must be a function");
            }
            if (arguments.length > 2) {
                thisArg = arguments[2];
            }
        }
        const len = toLength(items.length);
        const arr = isCallable(C) ? Object(new C(len)) : new Array(len);
        let i = 0;
        
        while(i < len) {
            iValue = items[i];
            if (mapFn) {
                arr[i] = typeof thisArg === 'undefined' ? mapFn(iValue, i) : mapFn.call(thisArg, iValue, i);
            } else {
                arr[i] = iValue;
            }
            i++;
        }
        return arr;
    }
})();

module.exports = Array;
```

## 测试代码

``` javascript
const Array = require('./index');

describe('Array.from', () => {

    describe('about first arguments', () => {
        test('if first arguments is string return this arr', () => {
            expect(Array.from('123')).toEqual(['1','2','3'])
        })
        test('if first arguments is null throw error', () => {
            expect(() => Array.from()).toThrow();
        })
    })

    describe('about second arguments', () => {
        test('if has second arguments and it is not function throw error', () => {
            expect(() => Array.from([1,2,3], 123)).toThrow();
        })
        
        test('return the double arr', () => {
            expect(Array.from([1,2,3], x => x*2)).toEqual([2,4,6]);
        })

        test('return the arr 1,2,3,4,5', () => {
            expect(Array.from({length: 5}, (v, i) => i+1)).toEqual([1,2,3,4,5]);
        })
    })

    describe('about this third arguments', () => {
        test('if has third arguments bind it into mapFn', () => {
            const obj = {
                handle: x => x * 3
            }
            expect(Array.from([1,2,3], function(x){ return this.handle(x) }, obj)).toEqual([3,6,9]);
        })
    })
})

```
