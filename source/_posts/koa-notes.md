---
title: Koa学习笔记
categories:
  - 学习笔记
date: 2018-08-20 17:15:14
tags:
- koa
- nodejs
---

## Koa是干啥的

* 一个基于node.js平台的web开发框架。
* 由Express原班人马打造。
* 没有捆绑任何中间件，更小。

## api

### 一个无作用的Koa程序被绑定到3000端口

``` javascript
const Koa = require('koa');
const app = new koa();

app.listen(3000);
```
<!--more-->

### app.context

app.context是ctx的原型。可给app.context添加其他属性。
ctx上的许多属性都适用getter，setter和object.defineProperty()定义的。

``` javascript
const main = ctx => {
  ctx.response.body = 'Hello World';
};

app.use(main)
```

ctx的属性：

``` javascript
ctx = {
    response: {
        type: 服务端返回类型，默认text/plain。可指定返回类型。
        body: 指定的服务端返回内容。
    },
    request: {
        accepts: 客户端希望接受的返回类型
        path: 用户请求时的路由
    },
    cookies: {
        get: () => {}
        set: () => {}
    }
    
}
```

### next

当一个中间件调用 next() 则该函数暂停并将控制传递给定义的下一个中间件。

``` javascript
const Koa = require('koa');
const app = new Koa();

const one = (ctx, next) => {
  console.log('>> one');
  next();
  console.log('<< one');
}

const two = (ctx, next) => {
  console.log('>> two');
  next();
  console.log('<< two');
}

const three = (ctx, next) => {
  console.log('>> three');
  next();
  console.log('<< three');
}

app.use(one);
app.use(two);
app.use(three);

app.listen(3000);

```

以上输出>> one、>> two、>> three、<< three、<< two、<< one

### 抛出错误

#### 抛出505错误: Internal Server Error

``` javascript
const main = ctx => {
    ctx.throw(500)
}
```

#### 抛出404错误: Page Not Found

``` javascript
const main = ctx => {
    ctx.response.status = 404;
    ctx.response.body = 'Page Not Found'
}
```

#### 处理错误的中间件：让最外层的中间件统一处理所有中间件的错误处理。

``` javascript
const handler = (ctx, next) => {
    try {
        await next()
    }
    catch (e) {
        ctx.response.status = err.statusCode || err.status || 500;
        ctx.response.body = {
            message: err.errorMessage
        }
    }
}
const main = ctx => {
    ctx.throw(500)
}

app.use(handler);
app.use(main);
```

localhost:3000页面中显示{message: 'Internal Server Error'}

#### error时间的监听

app.on('error',(err,ctx) => { })可以用来监听运行中的错误。

``` javascript
const main = ctx => {
    ctx.throw(500)
}
app.on('error', (err, ctx) => {
    console.log(err);
})
```

## koa提供的模块

### koa-route

koa-route可以用来替代原生路由ctx.request.path

``` javascript
const route = require('koa-route');
app.use(route.get('/'), main);
app.use(route.get('/about', about));
```

### koa-static

koa-static封装了静态资源的请求。为（图片、字体、样式表）一个一个写路由很麻烦，所以需要统一封装。

``` javascript
const path = require('path');
const serve = require('koa-static');

const main = serve(path.join(__dirname));
app.use(main);
```


### koa-compose

koa-compose模块将多个中间件合成为1个。
 
``` javascript
const compose = require('koa-compose');
const logger = (ctx, next) => {
    console.log('logging....');
    next();
};
const main = ctx => {
    ctx.response.body = 'hello world';
};
const middleware = compose([logger, main]);
app.use(middleware);
```

### koa-body

koa-body模块可以用来从POST请求体中提取键值对

``` javascript
const koaBody = require('koa-body');
const mian = ctx => {
    const body = ctx.request.body;
    if (!body.name) ctx.throw(400, '.name required');
    ctx.body = { name: body.name };
}
app.use(koaBody());
```

运行：

```
$ curl -X POST --data "name=Jack" 127.0.0.1:3000
{"name":"Jack"}
```

### koa-basic-auth

用于用户登录认证。弹出提示框用户名密码，输入正确返回正确对应的body内容。

``` javascript
const auth = require('koa-basic-auth');
app.use(auth({name: 'xlj', pass: 'xljpass'}));
```

在请求体中设置`Authorization: Basic + base64转换之后的用户名密码`

{% asset_img 1.jpg This is an example image %}

### koa-views

views(要渲染的文件path，配置obj)
map: 匹配每一个.html结尾的文件都要用nunjucks模板引擎去渲染。

``` javascript
const views = require('koa-views');

app.use(views(__dirname, { map: {html: 'nunjucks' }}))
 
// render `user.html` with nunjucks
app.use(async function (ctx) {
  await ctx.render('user.html')
})
```

### koa-session

koa-session中间件默认是基于cookie实现的，并且可以通过传入options.store使用扩展的存储方式(redis,mongoDB)

基于cookie实现的session有缺点：

* session没有加密就存储在客户端
* 浏览器中的cookie总是有长度限制

### koa-csrf

作用：用于项目安全加固。[Koa2项目安全加固建议--完整版](https://cnodejs.org/topic/5a502debafa0a121784a89c3)

原理：在你的提交表单里增加一个独有的临时的令牌用于服务器端验证，从而保证提交数据的正确性，防止CSRF攻击

CSRF：跨站请求伪造


## 参考文档

[Koa 框架教程](http://www.ruanyifeng.com/blog/2017/08/koa.html)
[Koa examples](https://github.com/koajs/examples)







