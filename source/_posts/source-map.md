---
title: 深入理解Source Map
categories:
  - 深入探究
date: 2018-09-06 17:49:44
tags:
---

## 背景
 
我们的项目源码通常通过一系列转换才能投入生产。但是，这也意味着，最终生成的代码已经改头换面了，不方便调试。

常见的源码转换，主要是以下三种情况

* 压缩，减小体积
* 多个文件合并，减少http请求数
* 通过loader编译语言，例如TypeScript转换到js

Source Map为我们解决了这一问题。Source Map能将打包后文件映射到源文件中的原位置，方便开发者debug。

## 什么是Source Map

Source Map是一个信息文件，里面存储着位置信息，也就是转换之后的代码的每一个位置，所对应的转换前的位置。debug的时候，可以直接显示源代码，而不是转换后的代码。

<!--more-->

### Source Map文件构成

一个简单的例子：

``` json
{
    version: 3,
    file: "script.js.map",
    sources: [
        "app.js",
        "content.js",
        "widget.js"
    ],
    sourceRoot: "/",
    names: ["slideUp", "slideDown", "save"],
    mappings: "AAA0B,kBAAhBA,QAAOC,SACjBD,OAAOC,OAAO..."
}
```

* version：Source Map的版本，目前为3
* file：转换后的文件名
* sources：含所有源文件名的数组
* sourceRoot：(可选)源文件所在目录
* names：包含源文件中所有的变量和函数的数组
* mappings：一个包含实际代码映射的Base64 VLQ字符串，也就是记录位置信息的字符串（哈！Source Map神奇的点就在于这里）


#### mappings属性

下面就来讲讲Source Map最神奇的地方: **mappings**属性，它体现了两个文件的各个位置是如何一一对应的。

{% asset_img 15362018507952.jpg This is an example image %}

mappings属性是一个字符串，分成三层：

* 第一层是行对应：以分号**；**表示。第m个分号前的内容，就对应源码的第m行。
* 第二层是列对应：以逗号**，**表示。第m行中第n个逗号前的内容，就对应源码的第m行n列的位置。
* 第三层是位置转换，以VLQ编码表示，代表该位置对应的源码位置。

```
mappings: 'AAAAA,BBBBB;CCCCC'
```
以上就表示，转换后的代码分两行，第1行2列，位置信息分别是AAAAA,BBBBB，第2行1列,位置信息只有CCCCC。

每个位置使用5位，比如第1行第1列位置信息是AAAAA，它的每一位都代表了一个信息，能把转换后的具体某列的位置映射到sources中的具体某个源码文件中的具体行具体列，甚至names中的具体的某个变量。

* 第一位，表示这个位置在转换后代码的第几列
* 第二位，表示这个位置属于sources属性中的第x个文件
* 第三位，表示这个位置属于源码的第x行
* 第四列，表示这个位置属于源码的第x列
* 第五位，（可省）表示这个位置属性names属性中的第x个变量

每一位都采用VLQ编码表示；由于VLQ编码是变长的(数值在-15-15之间，用一个字符表示；超出这个范围用多个字符表示)，所以每一位都可以由多个字符构成。

对于AAAAA来说：每个位置都是A，A在VLQ中表示o，因此他的意思是，该位置在转换后代码的第o列，属于sources属性中的第o个文件，属于源码中的第o行o列，对应names属性中的第o个变量。

了解更多[VLQ编码](https://en.wikipedia.org/wiki/Variable-length_quantity)。

### 工作原理

通过向转换后文件底部添加以下特殊注释，向浏览器说明Source Map可用(前提是浏览器如果可以支持Source Maps)，打包后的每一个文件都可指定一个Source Map文件。Source Map文件中通过mappings属性记录了转换后代码和源码的位置一一映射。

    //# sourceMappingURL=/path/to/script.js.map

### 如何生成Source Map

webpack中提供的devtool属性可以配置生成Source Map。除此之外，还有以下的几种方式可了解下：

* UglifyJS：允许合并压缩，同时支持生成Source Map的命令行标志

```  
uglifyjs [input files] -o script.min.js --source-map script.js.map --source-map-root http://example.com/js -c -m
```
* [Closure](http://www.html5rocks.com/en/tutorials/developertools/sourcemaps/#toc-howgenerate)
* [CoffeeScript Compiler](https://coffeescript.org/#source-maps)
* [GruntJS Task for JSMin](https://github.com/twolfson/grunt-jsmin-sourcemap)

本文主要是通过webpack中的配置来说明Source Map的不同类型。更多方式可看[这篇文章](https://code.tutsplus.com/tutorials/source-maps-101--net-29173)。

``` javascript
// 通过webpack配置文件设置Source Map
devtool: "source-map"
```
## Source Map类型

在webpack配置文件中，给devtool属性设置不同的Source Map类型，不同的类型值会极大影响打包和构建速度。

以下是官网列出的类型以及对应的构建和重构速度对比，以及质量对比。


{% asset_img 15361374592457.jpg This is an example image %}
开发环境和生产环境需要对应不同的值。在开发环境中，可牺牲包大小来达到快速生成Source Map；在生产环境中，需要精确的支持最小粒度的独立Source Map。

下面详细说明下devtool值对应的quality，并以此来判断适用环境。

### quality

* bundled code：各个模块没有彼此分离，是一个合并之后的超大大的js。
* generated code：各个模块彼此分离，并用模块名注释。看到的模块是通过webpack处理之后的。

    ``` javascript
    // 源代码
    import {test} from "module"; test();
    // generated code
    var module__WEBPACK_IMPORTED_MODULE_1__ = __webpack_require__(42); module__WEBPACK_IMPORTED_MODULE_1__.a();.
    ```

* transformed code：各个模块彼此分离，并用模块名注释。看到的模块是loaders处理之后，webpack处理之前的代码

    ``` javascript
    // 源代码
    import {test} from "module"; 
    class A extends test {}
    // transformed code
    import {test} from "module"; 
    var A = function(_test) { ... }(test);
    ```

* original source: 每个模块彼此分离，并用模块名注释。可以看到模块是转换前的，和源码一致。但是，这个需要loader支持，如果某个模块经过某个loader没有正确处理Source Map,那生成的代码是无法映射到源码上的。

     ``` javascript
    // 源代码
    import {test} from "module"; 
    class A extends test {}
    // original source
    import {test} from "module"; 
    class A extends test {}
    ```

* without source content：Source Map不包含源码内容中，浏览器通常会尝试从web服务器活着文件系统加载源码。此时必须确保设置了正确的output.devtoolModuleFilenameTemplate 。
* lines only：Source Map被简化为每行1个映射。会影响在行中设置断点调试。与压缩后的代码组合后，映射关系不可能实现，因为压缩之后只会输出一行。

### 开发环境

以下选项在开发环境中使用 较优


| 类型 | 说明 | quality |
| --- | --- | --- |
| eval | 每个模块都会用eval()执行，同时包含//@sourceURL指向Source Map。优点：快；缺点：不能正确地显示行号，因为它被映射到打包之后的代码而不是源代码(从loaders过来之后就没有Source Map了) |generated code|
|eval-source-map|每个模块都会用eval()执行，SourceMap会以DataUrl的形式放入到eval中。最初它很慢，但是它提供了快速重建速度并产生了真实文件。提供了最高质量的Source Map|original source|
|cheap-eval-source-map|每个模块都会用eval()执行；因为有cheap，所以没有列映射，只有行映射；忽略了源代码，只会显示loaders转换之后的代码|transformed code(lines only)|
|cheap-module-eval-source-map|每个模块都会用eval()执行；因为有cheap，所以没有列映射，只有行映射；因为有module，通过loaders处理之后获取了更好的结果|original source (lines only)|

### 特殊场景

以下选项在开发和生产环境中都不理想，除非是一些特殊场景，例如：在第三方工具中被使用。


| 类型 | 说明 |  
| --- | --- | 
| inline-source-map | 源代码作为数据包添加到包中 |  
| cheap-source-map |无列映射，忽略loader|
|inline-cheap-source-map|类似于cheap-source-map，同时将源代码作为数据包添加到包中|
|cheap-module-source-map|无列映射，简化了loader到单行映射|
|inline-cheap-module-source-map|类似cheap-module-source-map，同时将源代码作为数据包添加到包中|

### 生产环境

以下选项适合在生产环境中使用


| 类型 | 说明 | quality |
| --- | --- | --- |
| none | 没有源代码 | bundled code |
|source-map|单独生成对应的.map文件。同时在包中引用注释，让开发工具知道在哪里找到它|original source|
|hidden-source-map|类似于source-map，但是不引用注释，在你只不希望向开发工具暴露源文件，只想错误堆栈跟踪，报告错误的时候使用|original source|
|nosources-source-map|在没有源代码内容的情况下创建，可以用来不公开源代码，只用来堆栈跟踪|without source content|


## 结合实践来验证Source Map类型的使用

可[参考源码](https://github.com/fechengxiaocheng/my-source-map-demo)，在本地修改webpack.prod.js中的devtool的属性，执行npm run build来验证打包之后的Source Map(打包后的文件在dist文件夹中)。在浏览器中click按钮，看console中的报错信息。

以下则是不同Source Map类型的调试信息。

### none

没有展示报错代码所在的源文件。
{% asset_img 15361297824898.jpg This is an example image %}


### eval

* 每个源文件都用eval包裹
* 在每个文件的末尾加上//# souceURL源码路径
* 不产生souce-map文件

{% asset_img 15361290231211.jpg  This is an example image %}

{% asset_img 15362045957806.jpg This is an example image %}


包含eval关键字的配置项并不单独产生.map文件（eval模式有点特殊，它和其他模式不一样的地方是它依靠sourceURL来定位原始代码，而其他所有选项都使用.map文件的方式来定位）

### source-map

* 产生map文件
* 每个bundle文件末尾都会跟上sourceMappingURL对应源文件的信息

点击click之后，直接显示报错的源文件print.js
{% asset_img 15361296678332.jpg This is an example image %}


### hidden-source-map

打包后的app.js与source-map的相比，少了结尾处指向.map文件的注释。但是output目录中的index.js的app.js.map没有少

因此指向的非报错的实际源文件，而是打包之后的大bundle文件。
{% asset_img 15361297172789.jpg This is an example image %}



### cheap

* 有.map文件
* .map文件不包含列信息。



不加cheap：
{% asset_img 15354435655251.jpg This is an example image %}
加cheap可以看到无列信息，鼠标没有自动定位到：
{% asset_img 15354442606431.jpg This is an example image %}



### eval-source-map

* 有eval包裹代码
* 把sourcemap当成datauURL引入（会导致bundle文件加大）

{% asset_img 15361302940330.jpg This is an example image %}

{% asset_img 15361301710207.jpg This is an example image %}


### cheap-source-map

和source-map生成结果差不多。但是cheap-source-map比source-map生成的.map文件少了列信息。

* 有map文件
* bundle文件末尾有sourceMappingURL
* 不包含列信息
* 不包含loader的sourcemap


可以看到.map文件中mapping中的字符用；分隔。(eval-source-map用，分隔)

* `，`表示列分割
* `;`表示行分割

所以有cheap的话就没有`，`了。

    "mappings":"AAAA"

不包含loader的sourcemap是指，例如使用了loader转化，debug到就是定位到编译后的代码，而不是源码。

{% asset_img 15361305741911.jpg This is an example image %}



### cheap-module-source-map
* 包含loader的sourcemap

{% asset_img 15361307862136.jpg This is an example image %}


带cheap的可以提高打包速度，但是在浏览器中只能对应到具体的行，而不能对应到具体的列，对调试带来不便



### cheap-module-eval-source-map

最优

5467ms

{% asset_img 15361308360533.jpg This is an example image %}

既能加快打包速度(cheap),又能对应到具体的源文件(eval-source-map),又能包含loader(module)


## 参考文档

[Devtool官网](https://webpack.js.org/configuration/devtool/#src/components/Sidebar/Sidebar.jsx)
[An Introduction to Source Maps](http://blog.teamtreehouse.com/introduction-source-maps)
[Webpack devtool Source Map](http://cheng.logdown.com/posts/2016/03/25/679045)
[在webpack中使用devtool详解](http://www.php.cn/js-tutorial-401287.html)
[Webpack中的sourcemap](https://www.cnblogs.com/axl234/p/6500534.html)
[阮一峰JavaScript Source Map 详解](http://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html)
[在生产和开发环境中合理设置source-map类型](https://blog.csdn.net/liwusen/article/details/79414508)








