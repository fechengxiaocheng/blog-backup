---
title: 【解读源码系列】Array.prototype.includes
categories:
  - 源码系列
date: 2018-09-11 18:21:27
tags:
---

判断数组中是否含有某个值，返回true/false

## 用法

arr.includes(searchElement)
arr.includes(searchElement, fromIndex)

* searchElement：需要查找的元素值
* fromIndex: 从某个索引开始查找(可选)。如果为负，从array.length + formIndex处开始查，并跟0比取最大值。默认为0.

## 思路

* formIndex如果大于等于数组长度Array.length, 不查下去，return false。
* 数组长度AArray.length为0, 不查下去，return false。
* 设置开始循环判断的k值，默认为k为formIndex || 0，若formIndex小于0，k取formItem+Array.length和0相比的较大值。
* 从k处循环，直到k >= length跳出循环。判断如果数组中有值和searchElement相等，return true。

## 实现代码

``` javascript
Object.defineProperty(Array.prototype, 'includes', {
    value: function(searchElement, fromIndex) {
        if (this === null) {
            throw new typeError('"this" is null or undefined');
        }
        const o = new Object(this);
        const length = o.length;
        fromIndex = fromIndex || 0;
        if (fromIndex >= length) {
            return false;
        }
        if (length === 0) {
            return false;
        }
        let k = fromIndex;
        if (fromIndex < 0 ) {
            k = Math.max(fromIndex + length, 0);
        }
        while(k < length) {
            if (o[k] === searchElement) {
                return true;
            }
            k++;
        }
        return false;
    }
})
```

## 测试代码

``` javascript
const includes = require('./index');
test('if formIndex greater than length , return false', () => {
    expect([1,2,3].includes(1, 100)).toBeFalsy();
})

test('if length of array === 0, return false', () => {
    expect([].includes(1)).toBeFalsy();
}) 

test('formIndex < 0', () => {
    expect([1,2,3].includes(1, -100)).toBeTruthy();
    expect([1,2,3].includes(3, -2)).toBeTruthy();
    expect([1,2,3].includes(1, -2)).toBeFalsy();
})

test('[1,2,3] includes 1', () => {
  expect([1,2,3].includes(2, 1)).toBeTruthy();
})

```



