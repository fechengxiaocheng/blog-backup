---
title: 面试要点查漏补缺
categories:
  - 学习笔记
date: 2019-03-20 17:52:06
tags:
---

# 面试要点查漏补缺

[接上篇面试总结](./review-think.md)

##  JS基础

* null和undefined区别
* 字符编码(ASCII,Unicode,UTF*8,Base64)
* 闭包（定义、作用） 
* 运算符 对象相加过程
* 介绍下原型链（解决的是继承问题吗）
* 标准库toString()、Object.getOwnPropertyDescriptor(obj, 'a')、Object.defineproperty()
* 数组
    * push,pop,unshift,shift,reverse,splice,sort
    * contact,slice,map,forEach,every,some,reduce
    * 数组复制
    * 类数组转化为数组（延伸：类数组对象有哪些？nodelist和htmlcollection有哪些区别）
* 面向对象
    * new（实现过程）
    * 不加new后果？如何避免？
    * Object.create()的理解
    * this的实质?javascript为什么要设计this?
    * apply、call、bind手写代码
    * 对象的拷贝
* 严格模式，用处
* setTimeout和setInterval运行机制
* js实现类的继承
* 如何实现深拷贝：1、JSON.parse(JSON.stringify(arr)) 、2、jquery的extends({}, {a:1}) 3、自己用递归实现
* 6种跨域方法
* 正则
    * 零宽断言(?=pattern)、负前瞻(?!pattern)
    * 捕获和非捕获（?:）（(?<xlj>abc)组名为xlj）
    * 贪婪和非贪婪(默认都是贪婪的，后面加？就是非贪婪。+？、*？、{2,5}?尽可能的少)
    * 反向引用 'aabbccddee'.match(/(\w)\1/g)
    * 反义 ``[^aeiou]、\W \S \D \B``
* 常见设计模式7种。工厂模式、单例、策略、装饰者、适配器模式、代理模式、中介者模式、发布/订阅和观察者模式区别在哪，各自用在哪里。
* es6
    * 手写promise、手写Promise.all()，手写Promise.race()
    * promise、async、Generator有什么区别 
    * 对async、await的理解，内部原理
    * ES6中的map和原生的对象有什么区别
* 缓存
    * service worker离线缓存应用解决方案
    * 了解pwa原理
* 事件代理原理?如何自己实现？优缺点？


## 浏览器、网络

* 前端性能优化
    * 浏览器输入URL到页面呈现在面前发生了什么？每一个步骤能性能优化的点。
* 从浏览器多进程以及js单线程 深入了解完整的js运行机制
    * 进程与线程关系和区别？
    * 浏览器有哪些进程？
    * 浏览器多进程优势？
    * 渲染进程中有哪些线程？
    * 主进程与渲染进程两个进程之间是如何通信的
    * 梳理浏览器内核（渲染进程）中各个线程的管理
* http
    * http特点、常见首部字段（通用首部、请求首部、相应首部、实体首部）、常见状态码（200、204、205、206、301、302、304、401、404、500、501、503）
    * post和get区别
    * https含义（http+ssl）、在哪一层实现的、解决了http存在的哪些问题（加密+证书+不完整的内容（中间人攻击））、加密方式
    * http协议定义的四种常见数据post方式
        www-form-ulrencoded
        multipart/form-data
        application/json
        text/html 
    * 介绍http2.0
    * http1.1时如何复用tcp连接
    * 通过什么做到并发请求

* 负载均衡（通过ngix反向代理来实现，解决高并发问题，把请求通过服务器集群中的某台调度到其他的服务器中） 和 cdn（内容分布网络，1、就近，2、减少根服务器压力）
* http2解决了哪些问题（1、多路复用（解决chrome同域最多只能6次同时请求的问题，开一个tcp，多个请求）2、header压缩）
* http3: 基于udp，但是收纳了tcp的精华。快有可靠
* 遇到过哪些兼容性问题
* dns解析过程
* 前端性能数据如何采集
* 前端错误监控怎么做？（window.onerror的跨域问题怎么解决？错误准确位置需要用到source-map）
* 浏览器event loop
* reflow 和 repaint
* 项目中如何处理安全问题



## 算法coding题

* 数组去重怎么实现？（多种方法：set、filter、includes、indexOf、splice、sort、对象属性不能一样）
* 一串数字千位分隔，正则。
* 1元一瓶水，2个空瓶换一瓶水, 20元换几瓶水? 递归实现
* 给你一个能随机返回1-7的函数，如何实现能随机返回1-11的函数
* 一堆硬币、只有一个假硬币，一个天平，最少多少次可以确定？三分搜索
* 常见排序5种：冒泡（冒泡排序如何优化）、桶排序、选择排序、插入排序（扑克牌）、快速排序（二分）
* 栈、队列、二叉树、二分搜索树（所有左节点比右节点值小）、AVL
* 深度优先、广度优先(利用队列)
* 实现函数节流和函数防抖，并阐述他们的区别与使用场景
* sum(2, 3)实现sum(2)(3)的效果
* 手写两个大数相加
* 手写二叉树层序遍历
* 【coding】JS的全排列(10分钟)


## react、vue

* react挂载和生命周期流程。首先继承react.component，每个组件实例都有render方法，render实际是用了react.createElement方法，生成实例的js对象{type:xxx,key:xxx,props: {children: {}}},在组建进行挂载过程之前，组建实例都只是js对象。然后开始reactDOM.render() 挂载，进入到合成组建的生命周期（另外三种组建都没有生命周期，node=object、string／number、null。生命周期分为mounting和mounted两种过程。另外，当出发setState的时候，事务和状态共同构成了组建的更新（当isBatching策略为false的时候，把setState回调放入dirtyComponent，最后更新的时候一次性合并更新）。如果在componetwillupdate和componentshouldupdate中执行setstate，会造成死循环，因为那会儿触发的时候pendingstatequeue变为true，变为true之后又会触发updatecomponent，而updatecomponent又会执行componetwillupdate／componentshouldupdate，造成死循环。
* virtural dom理解、dom diff理解
* 传统diff算法是如何运算的，为什么是O（n3）
    ![](media/15511645960307/15514225848346.jpg)
* react diff算法
    * tree diff
    * component diff
    * element diff （为什么加唯一id就可以优化算法？具体是怎么操作的 还原新老两棵树差异的最步骤） 
* 介绍react优化
    * 从diff算法角度：尽量不要改变dom层级的移动、相同类型的组建不会有同样的dom结构、key不要用index用id。
    *  shouldcomponentupdate、React.PureComponent
    *  纯函数
    *  Fiber架构（算法、服务端渲染、异步渲染成为可能）
* 介绍Fiber，
    * 生命周期：16.X声明周期的改变、16.X中props改变后在哪个生命周期中处理
    * 服务端：rendertoString变成rendertoNodeStream；reactDOM.render变成reactDOM.hydrate（实现脱水和注水）
    * 异步渲染：让react异步渲染（分时间段渲染，优先紧事件处理）成为可能（未来两件大事：suspense（用同步的代码实现异步的操作），hooks（useState方法传入初始值））
* react.fragment的理解，在哪种场景下使用
* 高阶组建的理解、有何作用、使用场景（redux的connect）、如何自己实现一个高阶组建。mixins为什么会被替代？有哪些问题？
* 组建懒加载Bundle，原理
* Mvvm是什么，和MVC区别？
* react和vue之间的区别
* 合成事件的理解
* super(pros)作用，super()一定要调用么？props一定要传么？ 
* 单元测试：enzyme jest
* react异步渲染的概念,介绍Time Slicing 和 Suspense
* 介绍纯函数、介绍JSX、介绍高阶组件
* 如何解决props层级过深的问题
* 服务端渲染SSR
* react-router如何配置、原理（当前的path和pathnamke匹配规则，有一个router-regexep包)、前端路由是如何跳转的、路由的动态加载模块、介绍路由的history、嵌套留有（可以讲讲4之前和4之后版本不同，4之后仅仅是组件）、动态路由${this.props.match.url}
* 介绍redux数据流的流程，主要解决什么问题、redux请求中间件如何处理并发、redux如何实现多个组件之间的通信、多个组件使用相同状态如何进行管理、使用过的redux中间件、接入redux的过程 、绑定connect的过程 、connect原理、你用到了哪些redux中间件（redux-promise，redux-logger）、中间件原理（对store.dispatch进行改造，在发出action之后，reducer之前添加其他的功能。其实就是dispatch功能的增强）、如何自己实现一个redux中间件、redux实现一个简单的计时器。
* vue
    * vue如何实现双向绑定
    
### node相关

* koa
    * 使用过的koa2中间件koa-compress，koa-router，koa-views，koa-bodypaser，koa-static、koa-multer
    * koa-bodyparser原理（请求报文中的content-type4种不同类型分别作出不同的处理，然后把json解码，url解码，解码结果挂载到ctx.request.body上）
    * 介绍2个自己写过的中间件
    * koa中response.send、response.rounded、response.json发生了什么事，浏览器为什么能识别到它是一个json结构或是html
    * express和koa区别
* pm2：处理node进程的
* cluster: 创建共享服务端口的子进程。单个nodejs单线程的，为了发挥cpou多核优势，需要启用一组去处理负载。
* node核心模块
* node单线程优点
* nodejs是如何做到I/O的异步和非阻塞的呢

## css、html
* css3有哪些新属性
* html5新特性
    * websocket协议了解
    * 新特性。canvas、音频视频radio video、localStorage和sessionStorage、webworker、websocket、html5语义化标签header、nav、footer、article、section
* 介绍css3种position：sticky
* 清除浮动
* transform动画和直接使用left、top改变位置有什么优缺点
* 动画的了解 
* flex
* 水平垂直居中
* CSS3中对溢出的处理
    display: -webkit-box;
    -webkit-box-orient: vertical;
    over-flow: hidden;
    text-overflow: ellipsis;
    -webkit-line-clamp: 2
* positioon sticky(黏性布局，position：sticky；top、：0)、unset（可继承属性，类似于inherit，对于非继承属性，类似于initial）
* bfc：定义的渲染规则：看这两篇就懂了[BFC与IFC概念理解+布局规则+形成方法+用处](https://segmentfault.com/a/1190000009545742)、[浅谈BFC和IFC](https://blog.csdn.net/mevicky/article/details/47008939)
    * box垂直方向一层一层排列
    * maigin垂直方向会合并
    * 计算高度的时候float也会算进去
    * 两个bfc互相独立
* 如何触发bfc：
    * overflow：非visible
    * dispaly：absolute，fixed
* bfc可以解决哪些问题：
    * 清除浮动
    * float：left两列布局
    * margin 不合并
* 如何清除浮动
    * 添加空div，clear：both
    * 父元素直接添加height
    * 父元素overflow：hidden，就会触发一个bfc，自动计算浮动元素高度
* 水平垂直居中方案
    * 1、absolute，top：50%，left 50%，transform: translate(*50%, *50%)
    * 2、dosplay：flex，align-items：center： justify-content：center
* 两列布局，左固定，右自适应
    * 1、absolute、margin*left：200px
    * 2、float：left over*flow：hidden
    * 3、display：flex，flex：1 
* css3如何实现动画，有啥区别
    * 1、transition：width 3s
    *2、animation：@keyframeg 
    
* 有哪些多屏适配方案
    * media query + rem
    * em和rem区别是啥？
    * 弹性布局

## 其他

* 区别ing
    * white-space和word-break和word-wrap区别
    * 伪类和伪元素的区别
    * exports和module exports、export和export default的区别
    * http长连接和websorket长连接的区别
    * npm install --save 和 npm install --save-dev的区别
* 对于AST的理解
* iconfont引入的方式 
* 文件上传如何做断点续传(html5的input type="file", 把几个G的文件首先用唯一md5标识（用来告诉后端是哪个文件的）用slice(0，10 * 1024 * 1024)每10MB分割，如果碰到网络问题，2s后尝试重新连接。后端拿到md5信息的文件的时候，可以从数据库中查找是否已经有这个文件了，并且告诉客户端，直接从第x个芬片开始传，就节省了前面x*1次的分片的传输了)





