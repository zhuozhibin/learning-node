## 1. 概述

模块能被用来组织大型的项目以及分发Node项目。Node应用由模块组成，采用CommonJS规范。

## 2. 模块引用

通过require方法引用模块到当前上下文。

```js
var http = require('http');
```
   
## 3. 模块定义

模块内定义的对象和方法默认都是模块私有的，必须导出才能给外部引用。导出需要用到module对象。

### 3.1 module对象

Node内部提供一个<font color=#A52A2A>Module</font>构建函数。所有模块都是<font color=#A52A2A>Module</font>的实例。

```js
function Module(id, parent) {
  this.id = id;
  this.exports = {};
  this.parent = parent;
  //...
```

每个模块内部都有一个<font color=#A52A2A>module</font>对象,代表当前模块，它有以下属性:

```js
module.id //模块的识别符，通常是带有绝对路径的模块文件名。
module.filename //模块的文件名，带有绝对路径。
module.loaded //返回一个布尔值，表示模块是否已经完成加载。
module.parent //返回一个对象，表示调用该模块的模块。
module.children //返回一个数组，表示该模块要用到的其他模块。
module.exports //表示模块对外输出的值。
```

### 3.2 module.exports属性

<font color=#A52A2A>module.exports</font>属性表示当前模块对外导出的接口，require方法引用模块实际上就是读取该属性(如下图)。

![image](https://raw.githubusercontent.com/zhuozhibin/learning-node/master/docs/module/exports.png)

该属性由模块系统创建，初始化值为:

```js
{}
```

模块将需要导出的接口挂载在exports对象作为属性即可，或者可以直接改变其值。

```js
//module a
module.exports.add = function (a, b) {
    return a + b;
};

//require a
var a = require('./a');
var sum = a.add(1, 2);
```

```js
//moudule a
var EventEmitter = require('events').EventEmitter;
module.exports = new EventEmitter();

setTimeout(function() {
  module.exports.emit('ready');
}, 1000);

//require a
var a = require('./a');
a.on('ready', function() {
  console.log('module a is ready');
});
```

### 3.3 exports变量

为了方便，Node为每个模块提供一个exports变量，指向module.exports。这等同在每个模块头部，有一行这样的语句：

```js
var exports = module.exports;
```

当要导出接口时，可以向exports对象添加方法：

```js
//module a
exports.sub = function(a, b) {
    return a - b;
}
```

由于exports默认是对module.exports的引用，若将exports变量直接指向一个新值，则会切断exports与module.exports的联系，导致使用exports不能导出接口：

```js
//module a
var EventEmitter = require('events').EventEmitter;
exports = new EventEmitter();//无效

setTimeout(function() {
  module.exports.emit('ready');
}, 1000);
```

```js
//module a
exports.hello = function() {
  return 'hello';
};

module.exports = {};//module.exports被赋予新值，导致exports无效
```

exports变量为了方便存在，但对于某些人却容易造成混乱，只使用module.exports则逻辑清晰许多。

## 4. 模块实现

Node引入模块的过程中，包含3步：

- 路径分析
- 文件定位
- 编译执行

Node中的模块主要分两类：

- 核心模块：Node自身提供的模块，已提前编译成而二进制执行文件，在Node进程启动时，部分核心模块被直接加载进内存。引入这部分核心模块时，在路径分析中优先判断，并且不用经过文件定位和编译执行的步骤，加载速度快。
- 文件模块：用户编写的模块，需要完成经历3个步骤。

require()方法首次加载模块时会进行缓存，对相同模块的二次加载一律采用缓存优先的方式。

### 4.1 路径分析

require()方法接受一个标识符参数，基于这个参数进行模块查找，参数主要包括以下几种类型

#### 4.1.1 核心模块

优先级仅次于缓存加载。

#### 4.1.2 路径形式的文件模块

包括相对路径和绝对路径形式文件，最终都转化成真实路径加载。

#### 4.1.3 非路径形式的文件模块

如connect这种非核心分路径形式的模块，其加载时按照从当前目录下的node_modules目录开始，逐级往上(父目录)查找node_modules目录，知道找到或者到根目录为止，这种方式的加载是最慢的。

### 4.2 文件定位

#### 4.2.1 文件扩展名

require()的标识符参数经常会出现不包含文件扩展名的情况，此时Node会安装.js、.json、.node的次序依次补足查找，所以为了方便与性能折中，如果是js文件则可以不加扩展名，如果是json和node则加上。

#### 4.2.2 目录分析(包)

可能按照扩展名依次补足也查找不到对于的文件，但是得到一个目录，Node会将其作为一个包处理。

Node会查找包目录下的package.json，取出其中main属性指定的文件名进行定位(可以不含扩展名，按扩展名分析查找)，如果没有main属性，则默认查找index文件(js,json,node)。