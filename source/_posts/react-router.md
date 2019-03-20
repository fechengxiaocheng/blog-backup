---
title: React Router v4 学习笔记
categories:
  - 学习笔记
date: 2018-09-13 14:25:46
tags:
---

为更加理解和掌握React Router v4，我在[github](https://github.com/fechengxiaocheng/react-router4.0-example)上创建了demo项目用来实践[React Router v4](https://reacttraining.com/react-router/web)。

## 介绍

``React Router v4``是一个完全使用``react``重写的``React``包，所有的东西都**仅仅是组件**。跟react router之前的版本理念都不一样了。

## 安装

``React Router`` 现在已经被划分成了3个包：``react-router``，``react-router-dom``，``react-router-native``。

不应该直接安装react-router包，这个包为``React Router`` 应用提供了核心的路由组件和函数，另外两个包提供了特定环境的组件（浏览器和 react-native 对应的平台），不过他们也是将 ``react-router`` 导出的模块再次导出。

本项目在web中应用，因此使用``react-router-dom``

``` 
npm install --save react-router-dom
```

<!--more-->

## Router

开始一个新项目时，决定要使用那种Router,``<BrowserRouter>`` or ``<HashRouter>``。

这两区别在于：

* BrowserRouter: 一般用来处理动态页面, 适合用户互动多的网站。

* HashRouter：一般用来处理静态页面，适合用于展示型的网站，seo的时候一般建议静态网页。地址栏中会多一个#(exp:http://localhost:3000/#/electionic)

我们的项目页面都是服务器动态生成的，因此我们的``router``使用``<BrowserRouter>``，在index.js中使用。

``` javascript
import { BrowserRouter } from 'react-router-dom';
<BrowserRouter>
    <App />
</BrowserRouter>, document.getElementById('root'));
```
### history对象

history对象由Router创建，用来保持对当前位置的追踪还有页面发生变化时候重新渲染页面。


### basename属性

页面如果部署在二级目录，可加加``basename='/xlj'``,
   
``` javascript
<BrowserRouter basename="xlj/">
```

打开的地址栏：http://localhost:3000/xlj/electionic

## Routes

``Routes``组件是``React Router``的重要组成部分。用来在路径符合的时候在某个地方渲染某个组件。

### 匹配路径

``React Router``包使用``path-to-regexp``包来判断当前path是否匹配当前的pathname。它的原理是：**将path字符串转换成正则表达式与当前的pathname进行匹配**。

当path与pathname匹配的时候，一个具有以下属性的**match**对象将会被作为 prop 传入：

* match.url : 当前的path与pathname相匹配的部分。Clothes.js中的path就是/clothes，pathname是/clothes/woman的时候，匹配的部分就是/clothes。
* match.path : 路由path。
* match.isExact : path === pathname。
* match.params : 一个包含着 pathname 被 path-to-regexp 捕获的对象。

### exact

下面代码块中要加exact，不加exact的话，下面的三个url都会匹配到第一行<Route path="/clothes" exact component={Woman} />中的path="/clothes"，即渲染的都是Woman组件。除非换顺序，把path='/clothes'换到最后面。

* http://localhost:3000/clothes
* http://localhost:3000/clothes/woman
* http://localhost:3000/clothes/man


```javascript
<Switch>
  <Route path="/clothes" exact component={Woman} />
  <Route path="/clothes/woman" component={Woman} />
  <Route path="/clothes/man" component={Man} />
</Switch>
```

### Route渲染组件内容

* 直接定义component属性

```
<Route path="/" exact component={Clothes} />
```

* 通过render渲染组件

```javascript
<Route path="/" exact render={(props) => (<Clothes {...props} data={{color: 'red'}} />)} />
```

这两种方法原理都是通过``React.createElement``来引入组件。一般情况用``component``就够了。但是通过``render``来定义可传入自定义属性。比如这里的``data``属性。 如果不用``render``，只有默认的``props``。

由routes渲染的组件默认有 ``props``，包含``match``，``location``，``history``(由router创建)

### 嵌套路由

4.0之前定义嵌套组件不是React的风格，是伪组件；4.0之后是一切皆组件。

4.0之前：

```javascript
<Router history={hashHistory}>
  <Route path="/" component={App}>
    <Route path="/repos" component={Repos}/>
    <Route path="/about" component={About}/>
  </Route>
</Router>
```

4.0之后：

本项目中``Clothes.js``和``Man.js``中分别嵌套了二级和三级路由。

本项目中``Fruites.js``中用路径参数捕获``/fruites``后面带的pathname部分。对于``/fruites/abc``的路径，都能匹配到`FruitesTemplate`组件。在`FruitesTemplate`组件中，使用`this.props.match.params.type`即可获取到对应的`abc`部分。

```javascript
<Route path="/fruites/:type" component={FruitesTemplate}/>
```

### 动态路由

``Clothes.js``中，用`this.props.match.url`代替`/clothes`，这样就不用写死了。

```javascript
<Switch>
    <Route path="/clothes" exact component={Woman}/>
    <Route path="/clothes/woman" component={Woman}/>
    <Route path="/clothes/man" component={Man}/>
    <Route path="/clothes/shoe" component={Woman}/>
    <Route path="/clothes/bag" component={Woman}/>
</Switch>
```

```javascript
// 动态路由
const url = this.props.match.url;
<Switch>
    <Route path={`${url}`} exact component={Woman}/>
    <Route path={`${url}/woman`} component={Woman}/>
    <Route path={`${url}/man`} component={Man}/>
    <Route path={`${url}/shoe`} component={Woman}/>
    <Route path={`${url}/bag`} component={Woman}/>
</Switch>
```

### 跳转

#### Link

```javascript
import { Link } from 'react-router-dom'
<Link to="clothes/bag">
<Link to={{ pathname: 'clothes/bag', search: '?id=1&code=2', hash:'#my-hash', state: {userName: xlj}}}>   
```
ps: 这里的state只在pathname === match url时才会有

#### NavLink

``NavLink``比``Link``有更丰富的``api``。可以在当前的``NavLink``上添加对应选中时候的样式``activeClassName``。

```javascript
import { NavLink } from 'react-router-dom'
<NavLink to="clothes/bag">
<NavLink to={{ pathname: 'clothes/bag', search: '?id=1&code=2', hash:'#my-hash', state: {userName: xlj}}}>
<NavLink to="clothes/woman" activeClassName="selected"> // 参考Electric.js
```

#### this.props.history

```javascript
this.props.history.push({
    pathname: '/xlj',
    search: '?id=1&code=2'
});
```

#### this.context.router.history

先当前组件中引入contextTypes属性，才能获取到context下的router。
    
```javascript
import PropTypes from 'prop-types';
    
clicktest = () => {
    this.context.router.history.push({
        pathname: '/xlj',
        search: '?id=1'
    })
}
    
purchasDetails.contextTypes = {
    router: PropTypes.object
}
```


## Switch

如下代码：加上``Switch``标签，在页面中只渲染出当前匹配的dom，匹配到一个之后就不忘下走了。不加``Switch``, 在http://localhost:3000/clothes/woman 中会渲染出两个``Woman``组件。

```javascript
<Switch>
    <Route path="/clothes" exact component={Woman} />
    <Route path="/clothes/woman" component={Woman} />
    <Route path="/clothes/woman" component={Woman}/>
</Switch>
```

## Redirect

``Redict``标签一般用于没有匹配到``path``的时候，默认跳转到的路径。

```javascript
<Switch>
    <Route path={`${url}`} exact component={Woman}/>
    <Route path={`${url}/woman`} component={Woman}/>
    <Route path={`${url}/man`} component={Man}/>
    <Redirect to={`${url}`} />
</Switch>
```

## 参考文档

[[译]简明React Router v4教程](https://juejin.im/post/5a7e9ee7f265da4e7832949c)
[4.0教程](http://reacttraining.cn/web/example/basic)
[React Router 4 简介及其背后的路由哲学](https://juejin.im/post/5a641747518825732d7fb25f)
