---
title: Dockfile学习笔记
date: 2018-07-18 18:23:13
categories:
- 学习笔记
---

## 是什么东西？

是一个脚本。
是创建了一个新镜像的脚本。
是基于基础镜像创建一个新镜像的脚本。


## 干啥用的？

创建新的docker镜像。在公司内部环境，需要基于公司内部的node镜像，指定公司内部的sass源和npm源，在指定的生产环境中运行项目。

<!--more-->

## 为啥要用？一定要用吗？

不用的话，无法在公司内部的测试机中运行项目。本地环境就不需要。

## 怎么用？

以下是某项目的的Dockfile文件示例(http://npm.xxx.com是公司内部源)

``` javascript
FROM registry.xxx.com/env/node:6.10.1-nginx

COPY package.json /app/package.json
WORKDIR /app

RUN npm uninstall
ENV SASS_BINARY_SITE http://npm.xxx.com/node-mirrors/node-sass/
ENV NODE_ENV production
RUN npm --registry http://npm.xxx.com install

COPY ./ /app/

RUN npm run product

EXPOSE 8080

CMD ["node", "server.js"]

```

### FROM

``` javascript
FROM registry.xxx.com/env/node:6.10.1-nginx

* FROM 指定创建新镜像的基础镜像；如果没有，则去Docker的公共库拉取下来。
* FROM 要在文件的第一行。
* 一个文件可以有多个From，如果要创建多个镜像的话。

```

### COPY

``` javascript
COPY package.json /app/package.json

```

* COPY 复制package.json到制定的文件中去/app/package.json

### ADD

``` javascript
ADD package.json /app/package.json

```

* ADD 复制package.json到制定的文件中去/app/package.json
* 用法与COPY相同，但是可以复制url作为源到指定的文件中。比COPY强大。

### WORKDIR

``` javascript
WORKDIR /app
    
```

* 切换目录到app,，相当于cd。

### RUN

``` javascript
RUN npm uninstall
RUN npm --registry http://npm.xxx.com install
RUN npm run product
    
```

* 在当前镜像的基础上执行指定命令，并提交为新的镜像。
* 后续RUN的命令在上一个RUN之后的镜像作为基础。

### ENV

``` javascript
ENV SASS_BINARY_SITE http://npm.xxx.com/node-mirrors/node-sass/
ENV NODE_ENV production
    
```

* ENV key value 指定一个环境变量，为后续的RUN指令使用。

### EXPOSE

``` javascript
EXPOSE 8080
    
```

* 指定要暴露哪个接口

### CMD

``` javascript
CMD ["node", "server.js"]

```

* 启动容器的时候执行。构建的时候不执行
* 一个容器只有一个CMD。如果有多个，只有最后一个生效。



