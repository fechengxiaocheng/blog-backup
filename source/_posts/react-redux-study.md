---
title: 深入理解react-redux
categories:
  - 深入探究
date: 2018-08-24 10:11:49
tags:
---

## redux和react-redux区别

待搞，先占坑。

## 理解redux中间件

### 什么是redux中间件

待搞，先占坑。

### 自己实现一个redux中间件

redux中间件的本质就是重写dispatch方法:

``` javascript
// 一个简单的logger中间件实现原理
store.dispatch = function dispatchLogger(action) {
    console.log('before action....');
    next(action);
    console.log('after action....');
}
```

通过redux-promise来实现``异步中间件``

``` javascript

// 一个promise异步中间件使用方法

// 1、返回一个action，action本身是promise
const actionCreatorA = (dispatch, title) => new Promise((resolve, reject) => {
    return fetch('xx/xx/xx').then(res=> {type: 'addTodo', payload: res.value});
})

// 2、返回一个action，action.payload是promise(需要从redux-actions模块引入createAction方法)
dispatch(createAction(
    'addTodo',
    fetch('xx/xx/xx').then(res=> res.value);
))

// 一个promise异步中间件实现原理
store.dispatch = function dispatchPromise(action) {
    // 如果action本身是promise，执行then，传入dispatch方法, 把异步请求到的action作为传入dispatch的入参。
    return isPromise(action) ? action.then(dispatch) : next(action); 
    // 如果返回的payload是promise，执行action.payload.then, 
    return isPromise(action.payload) ? 
        action.payload.then(res => {
            dispatch({...action, payload: res})
        }).catch(error => {
            return Promise.reject(error)
        }) : 
        next(action)
}
```
    
