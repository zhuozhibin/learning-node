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

<font color=#A52A2A>module.exports</font>属性表示当前模块对外导出的接口，require方法引用模块实际上就是读取该属性。

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
