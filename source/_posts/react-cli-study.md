---
title: react-koa-app-cli命令行工具
date: 2018-07-19 11:12:35
categories:
- 深入探究
tags: 
- react
- koa
- cli
---

## 基本思路

{% asset_img 1.jpg This is an example image %}

<!--more-->

## 知识点

### download-git-repo

用于在node中下载git仓库。

``` javascript
// 基本用法
const download = require('download-git-repo');
const url = 'github:fechengxiaocheng/react-koa-app';
download(url, target, (err) => {
    if (err) {
        // 处理下载错误的逻辑
    }
    else {
        // 处理下载完之后的逻辑
    }
})
```
    
- ``url``为要下载的地址

    * GitHub - github:owner/name or simply owner/name
    * GitLab - gitlab:owner/name
    * Bitbucket - bitbucket:owner/name

    项目中的``url``是github的``github:fechengxiaocheng/react-koa-app``，也可以简写``fechengxiaocheng/react-koa-app``

- ``target``要下载到的目标文件
    
    项目中的``target``为``my-app/.download-temp``临时目录

### ora

用于在终端打印某节点的对应信息。
    
``` javascript
// 基本用法
const ora = require('ora')
const spinner = ora('loading......').start();
setTimeout(() => {
    spinner.color = 'red'
    spinner.text = 'loading hello chengxiaocheng'
},1000)
setTimeout(() => {
    spinner.succeed(['succuss loading...']);
},2000)
```
    
项目中ora中来打印git下载进行、成功的信息  
    
### glob

使用patterns来匹配对应的文件

``` javascript
// 基本用法
const glob = require('glob');
const list = glob.sync('lib/*.js'); // 遍历匹配lib/*.js lib下所有的js文件
console.log(list);
```
    
### nodejs-path

path.basename(path[, ext])返回路径最后的一部分

``` javascript
path.basename('/foo/bar/baz/asdf/quux.html');
// 返回: 'quux.html'

path.basename('/foo/bar/baz/asdf/quux.html', '.html');
// 返回: 'quux'
```
    
path.resolve([...paths])把一个路径或路径片段的序列解析为一个绝对路径。

``` javascript    
path.resolve('/foo/bar', './baz');
// 返回: '/foo/bar/baz'

path.resolve('/foo/bar', '/tmp/file/');
// 返回: '/tmp/file'

path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif');
// 如果当前工作目录为 /home/myself/node，
// 则返回 '/home/myself/node/wwwroot/static_files/gif/image.gif'
```

fs.stat(filepath, (err, stats) => {stats.isDerectory()})
    
    * fs.stat是异步的,不能保证已执行完再去执行后面的，所以要放在回调中。
    
fs.statSync(filepath).isDerectory()     

    * fs.statSync是同步的，可以直接进行后面的操作
    
### inquirer

用来在终端与用户交互答案

``` javascript

const inquirer  = require('inquirer');

inquirer.prompt([
    {
        name: 'yourName',
        message: 'who are you?',
        default: 'chengxiaocheng',
    },
    {
        name: 'yourAge',
        message: 'how old are you?',
        default: '17',
    },
    {
        name: 'yourAdreess',
        message: 'where are you from?',
        default: 'shanghai',
    }
]).then(answers => {
    console.log('1...',answers);
    inquirer.prompt([
        {
            name: 'niceGirl',
            message: 'are you a nice girl?',
            type: 'confirm',
            default: true
        }
    ]).then(answers => {
        console.log('2...',answers)
    })
})
```
    
默认type为input，设置成confirm则确认y／n
    

![](media/15300065098836/15300823345173.jpg)

### metalsmith

一个简单的静态站点生成器,用来批量处理模板

### handlebars

模板引擎

``` javascript
const Handlebars = require('handlebars');
var source = "<p>Hello, my name is {{name}}. I am from {{hometown}}. I have " +
                "{{kids.length}} kids:</p>" +
                "<ul>{{#kids}}<li>{{name}} is {{age}}</li>{{/kids}}</ul>";
var template = Handlebars.compile(source);
    
var data = { "name": "Alan", "hometown": "Somewhere, TX",
                "kids": [{"name": "Jimmy", "age": "12"}, {"name": "Sally", "age": "4"}]};
var result = template(data);
console.log(result);
``` 
    
![](media/15300065098836/15300847850369.jpg)


使用handlebars模板引擎的插值方式{{xxx}}，把输入的内容插入到模板中对应的变量。

修改``creat-koa-app``中的``packge.json``,这里的变量名``projectName``，``projectVersion``，``projectDescription``与``creat-koa-app-cli``中的与用户在终端交互的值对应。

``` javascript
{
    "name": "{{projectName}}",
    "version": "{{projectVersion}}",
    "description": "{{projectDescription}}",
    ...
}
```



### rimraf

``node``中的``rimraf``等同于终端中的``rm -rf``

### log-symbols

用来在终端打印出好看的标示。

``` javascript
const logSymbols = require('log-symbols');
console.log(logSymbols.success, 'Finished successfully!');
console.log(logSymbols.info, 'Finished info!');
console.log(logSymbols.warning, 'Finished warning!');
console.log(logSymbols.error, 'Finished error!');    
```

![](media/15300065098836/15300886750173.jpg)


### chalk

在终端打印出自定义颜色，粗细，bgcolor

``` javascript
const logSymbols = require('log-symbols');
const chalk = require('chalk');
console.log(logSymbols.success, chalk.green('Finished successfully!'));
console.log(logSymbols.info, chalk.rgb(255,255,0).inverse('Finished info!'));
console.log(logSymbols.warning, 'Finished warning!');
console.log(logSymbols.error, chalk.red.bold('Finished error!'));    
```   
    
![](media/15300065098836/15300890284308.jpg)

### command

支持git风格的子命令处理

可以根据子命令自动引导到以特定格式命名的命令执行文件，文件名的格式是``[command]-[subcommand]``，例如：

macaw hello => macaw-hello

macaw init => macaw-init



