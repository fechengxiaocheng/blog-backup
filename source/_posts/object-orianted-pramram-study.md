---
title: 面向对象编程(封装&继承)
categories:
  - 深入研究
date: 2018-08-22 15:23:16
tags:
---

最近开始重新梳理js基础知识，边学习、边思考、边练手、边记录。[github源码地址](https://github.com/fechengxiaocheng/object-orianted-program-demo)


## 封装

### 构造函数模式

在构造函数中封装对象的属性和方法。

[demo1](https://github.com/fechengxiaocheng/object-orianted-program-demo/blob/master/demo1.js)

缺点: 原型对象中定义相同的属性或者方法时，每次新建实例都会生成一次，多占用内存。

思考: 能否将共有的方法只在内存中生成一次呢？

答案: 能。用原型模式

<!--more-->

### prototype模式

[demo2](https://github.com/fechengxiaocheng/object-orianted-program-demo/blob/master/demo2.js)

每个构造函数都有一个prototype属性，它指向一个对象，这个对象上面所有的属性和方法都会被实例继承。

#### prototype模式的验证

* isPrototypeOf: 某原型是否某对象的原型
    Cat.prototype.isPrototypeOf(cat1)
* hasOwnProperty: 某实例是否有自己的属性xx
    cat1.hasOwnProperty('name')
* in: 某实例是否拥有某属性
    'type' in cat2

## 继承-构造函数继承

### 构造函数绑定

[demo3](https://github.com/fechengxiaocheng/object-orianted-program-demo/blob/master/demo3.js)

在子类构造函数中，把父类构造函数绑定倒子类上。

``` javascript
Animal.apply(this, arguments);
```

### prototype模式

[demo4](https://github.com/fechengxiaocheng/object-orianted-program-demo/blob/master/demo4.js)

子类的构造函数链接倒父类的实例

``` javascript
Cat.prototype = new Animal();
Cat.prototype.constructor = Cat;
```

类的prototype被重新赋值之后，他的constructor也会被覆盖，导致继承链混乱(cat1明明是Cat构造的，却显示成了Animal)。所以要给constructor重新赋值下。

### prototype直接继承

[demo5](https://github.com/fechengxiaocheng/object-orianted-program-demo/blob/master/demo5.js)

只继承父类的公共属性和方法。

``` javascript
Cat.prototype = Aminal.prototype;
Cat.prototype.constructor = Cat;
```

* 优点：不用new实例，省内存
* 缺点：1、只能继承公有属性和方法。2、直接把父类原型赋值给子类原型，那么子类原型变动，也会引起父类原型改动。(子类原型上的constructor变成Cat, 父类原型的constructor也变成了Cat)

解决：利用空构造函数的实例

### 空对象中介

[demo6](https://github.com/fechengxiaocheng/object-orianted-program-demo/blob/master/demo6.js)

把空构造函数的原型链接到父类的原型；
把子类的实例链接倒空构造函数的实例；

把空对象作为中介，这样子类的原型改动不会牵连到父类，且空对象几乎不占用内存。

``` javascript
function F() {};
F.prototype = Animal.prototype;
Cat.prototype = new F();
```

### 拷贝继承

[demo7](https://github.com/fechengxiaocheng/object-orianted-program-demo/blob/master/demo7.js)

循环父类原型中所有的属性和方法，赋值到子类原型中。

``` javascript
for (let i in Parent.prototype) {
    console.log(i);
    Sub.prototype[i] = Parent.prototype[i];
}
```
## 继承-非构造函数继承

所谓非构造函数继承，就是a对象直接要继承b对象，而a、b对象不是构造函数，不能通过构造函数的方式继承。

### prototype链

[demo8](https://github.com/fechengxiaocheng/object-orianted-program-demo/blob/master/demo8.js)

创建空构造函数，把空构造函数的原型链接到父对象，返回这个空构造函数的实例赋值给子对象。即继承了空对象的所有属性和方法，而空对象原型是链接到父对象的，所以继承了父对象。


``` javascript
function F() {}
F.prototype = ParentObj;
SubObj = new F();
```

### 深拷贝&浅拷贝

[demo9](https://github.com/fechengxiaocheng/object-orianted-program-demo/blob/master/demo9.js)[demo10](https://github.com/fechengxiaocheng/object-orianted-program-demo/blob/master/demo10.js)

字符串的拷贝是拷贝值，对象的拷贝是拷贝内存地址。
浅拷贝只拷贝了对象的内存地址，而没有开辟新的栈。所以当子对象变，会影响父对象，因为都指向的是同一块内存地址。

浅拷贝思路：

循环父对象，把父对象所有的属性赋值给子对象。

深拷贝思路：

循环父对象，判断父对象的属性如果是对象、数组时分别返回全新的对象，把这个全新的对象赋值给子对象的属性。


## 参考文档

[Javascript面向对象编程](http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_inheritance.html)

[js实现深拷贝的方法](https://blog.csdn.net/sinat_36422236/article/details/79761648)
























