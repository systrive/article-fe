[返回首页](../../README.md)

# 浅谈前端模块化

## 背景

`JavaScript`程序本来很小，在早期，它们大多被用来执行独立的脚本任务，在Web页面实现简单的交互逻辑，一般不需要多大脚本。随着web2.0时代的到来，Ajax技术的广泛应用，React、Vue等前端库的出现，有了运行大量`JavaScript`脚本的复杂程序，还有一些被用在其他环境（例如 `Node.js`）。因此，有必要考虑将`JavaScript`程序拆分为可按需导入的单独模块的机制。

> 模块：实现特定功能的相互独立的一组方法。

因为有了模块，我们能更好的管理网页的业务逻辑，可以按照自己的需求去设计、使用各种模块。JS 模块规范有很多，目前比较流行的有：

* [CommonJS](https://javascript.ruanyifeng.com/nodejs/module.html)
* AMD
* CMD
* ES6 module

## CommonJS

### 概述

`Node.js` 是 CommonJS 规范的主要实践者。每个文件就是一个模块，有自己的作用域。在一个文件里定义的变量、函数、类都是私有的，对其他文件不可见。

### 基本用法

模块有四个重要的变量：
1. module：对象，代表当前文件模块
2. exports：对外暴露的接口
3. require：加载模块文件，基本功能是执行一个`JavaScript`模块，然后返回该模块的 `exports` 对象。
4. global：全局对象

例子：

```
// example.js
var x = 1
var addX = function (val) {
    return x + val
}
module.exports = {
    x, addX
}
// index.js
var example = require('./example.js')
console.log(example.x)       // 1
console.log(example.addX(1)) // 2
```

想要在多个文件分享变量，必须定义为`global`对象的属性，这种写法是不推荐的。

```
global.warning = true
```

CommonJS 模块特点如下：

* 所有代码都运行在模块作用域，不会污染全局作用域。
* 模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就被缓存了，以后再加载就直接读取缓存结果。想让模块再次运行，必须清除缓存。
* 模块加载的顺序，按照其在代码中出现的顺序。

### module 对象

Node 内部提供一个 `Module` 构建函数，所有模块都是 `Module` 实例。每个模块内部都有一个`module`对象，代表当前模块。有以下属性：

* module.id：模块的识别符，通常是带有绝对路径的模块文件名。
* module.filename：模块的文件名，带有绝对路径。
* module.loaded：返回一个布尔值，表示模块是否已经完成加载。
* module.parent：返回一个对象，表示调用该模块的模块
* module.children：返回一个数组，表示该模块要用到的其他模块。
* module.exports：表示模块对外输出的值。

根据 `module.parent` 是null 表示当前模块为入口脚本。

#### module.exports

`module.exports` 属性：表示当前模块对外输出的接口，其他文件加载该模块，实际就是读取 `module.exports`变量。

#### exports

`exports` 变量：每个 Node 为每个模块提供一个 `exports` 变量，指向 `module.exports`，这等同在每个头部都有这样的命令。

```
var exports = module.exports
```

对外输出模块接口时，可以向 `exports` 对象添加方法

```
exports.area = function (r) {
    return Math.PI * r * r
}
exports.circumference = function (r) {
    return 2 * Math.PI * r
}
```
注意：不能直接将 `exports` 变量指向一个值，因为这样等于切断了 `exports` 与 `module.exports` 的联系。

```
// 以下方法无效
exports = function (x) {
    console.log(x)
}
// 下面的`hello`无法对外输出。因为`module.exports`被重新赋值了。
exports.hello = function () {
    return 'hello'
}
module.exports = 'Hello World'
```

### require
#### require 加载规则

`require`命令加载文件，后缀名默认为 `.js`，根据不同路径寻找模块文件：
1. 如果参数字符串以 `"/"` 开头，则表示加载是一个位于绝对路径的模块文件。比如 `require('/home/user/project/foo.js')`
2. 如果参数字符串以 `"./"` 开头，则表示加载位于相对路径（与当前执行脚本位置相比）的慕课。比如 `require('./circle')` 将加载当前脚本同一目录的 `circle.js`。
3. 如果参数字符串不以 `"./"` 或 `"/"` 开头，则表示加载是一个默认提供的核心模块（位于 Node 的系统安装目录中），或者一个位于各级 `/node_modules` 目录的已安装模块（全局或局部安装）。
比如，`/home/user/project/foo.js` 执行了 `require('bar.js')` 命令，Node 会依次搜索以下文件。
```
/usr/local/lib/node/bar.js
/home/user/projects/node_modules/bar.js
/home/user/node_modules/bar.js
/home/node_modules/bar.js
/node_modules/bar.js
```
这样设计的目的使得不同模块可以将所依赖的模块本地化。

4. 如果参数字符串不以 `"./"` 或 `"/"` 开头，而是一个路径，比如 `require('example-module/path/to/file')` ，则通过步骤3先找到 `example-module` 的位置，再以它为参数，找到后续路径。
5. 如果指定的模块文件没有发现，Node 会尝试为文件名添加 `.js`、`.json`、`.node` 后，再去搜索。`.js` 文件会以文本格式的 `JavaScript` 脚本解析，`.json` 文件会以 JSON 格式的文本解析，`.node` 文件会以编译后的二进制文件解析。
6. 如果想得到 `require` 命令加载的确切文件名，使用 `require.resolve()` 方法

#### 目录的加载规则

通常我们会把相关文件放在一个目录，便于组织。这时最好为该目录设置一个入口文件，让 `require` 方法可以通过这个入口文件，加载整个目录。

在目录中防止一个 `package.json` 文件，并且将入口文件写入 `main` 字段。

```
// package.json
{
    "name": "my-project",
    "main": "./lib/my-project.js"
}
```

`require` 发现参数字符串指向一个目录后，会自动查看该目录的 `package.json` 文件，然后加载 `main` 字段指定的入口文件。如果 `package.json` 没有 `main` 字段，或者根本就没有 `package.json` 文件，就会加载该目录下的 `index.js` 文件或 `index.node` 文件。

#### 模块缓存

第一次加载某个模块时，Node 会缓存该模块。以后再加载该模块，就直接从缓存取出该模块的 `module.exports` 属性。

```
require('./example.js')
require('./example.js').message = 'Hello'
console.log(require('./example.js').nessage)    // "hello"
```
上面例子中，连续三次使用 `require` 命令加载同一模块，第二次加载的时候，为输出的对象添加了一个 `message` 属性。但是第三次加载的时候，这个 message 属性依然存在，说明`require` 没有重新加载模块文件，而是输出了缓存。

如果想要多次执行某个模块，可以让该模块输出一个函数，然后每次 `require` 这个模块的时候，重新执行一下输出的函数。

所有缓存的模块保存在 `require.cache` 之中，如果想要删除模块的缓存，可以这样写

```
// 删除指定模块的缓存
delete require.cache[moduleName]

// 删除所有模块的缓存
Object.keys(require.cache).forEach(function (key) {
    delete require.cache[key]
})
```

注意：缓存是根据绝对路径识别模块的，如果同样的模块名，但是保存在不同路径，`require` 命令还是会重新加载该模块。

#### 环境变量 NODE_PATH

Node 执行一个脚本时，会先查看环境变量 `NODE_PATH`。它是一组以冒号分隔的绝对路径。在其他位置找不到指定模块时，Node 会去这些路径查找。

* NODE_PATH 添加到`.babelrc`
* NODE_PATH 添加到 `package.json` `"scripts"` 中

```
// package.json
{
    "scripts": {
        "start": "NODE_PATH=lib node index.js"
    }
}
```

`NODE_PATH` 是历史遗留下来的一个路径解决方案，通常不应该使用，而应该使用 `node_modules` 目录机制。

#### 模块的循环加载

如果模块发生循环加载，即 A 加载 B，B又加载A，则B将加载A的不完整版本。

```
// a.js
exports.x = 'a1'
console.log('a.js: ', require('./b.js').x)
exports.x = 'a2'

// b.js
exports.x = 'b1'
console.log('b.js: ', require('./a.js').x)
exports.x = 'b2'

// index.js
console.log('index.js require a.js: ', require('./a.js').x)
console.log('index.js require b.js: ', require('./b.js').x)
```

上面三个 js 文件。其中 a.js 加载了 b.js，b.js 又加载了 a.js。这时，Node 返回 a.js 的不完整版本，所以执行结果如下：

```
$ node index.js
b.js:  a1
a.js:  b2
index.js require a.js:  a2
index.js require b.js:  b2
```

修改 index.js，再次加载 a.js 和 b.js

```
console.log('index.js require a.js: ', require('./a.js').x)
console.log('index.js require b.js: ', require('./b.js').x)
console.log('index.js require a.js: ', require('./a.js').x)
console.log('index.js require b.js: ', require('./b.js').x)
```
得到执行结果
```
$ node index.js
b.js:  a1
a.js:  b2
index.js require a.js:  a2
index.js require b.js:  b2
index.js require a.js:  a2
index.js require b.js:  b2
```

上面代码，第二次加载 a.js 和 b.js 时，会直接从缓存中读取 exports 属性，所以 a.js 和 b.js 内部的 console.log 语句就不会执行了。

#### require.main
`require` 方法有一个 `main` 属性，可以用来判断模块是直接执行，还是被调用执行。

直接执行的时候（node module.js），`require.main` 属性指向模块本身。

```
require.main === module
// true
```

调用执行的时候（通过 `require` 加载该脚本执行），上面表达式返回 `false`。

### 模块的加载机制

CommonJS 模块的加载机制：模块输入是被输出值的拷贝，也就是说一旦输出一个基本类型的值，模块内部变化就影响不到这个值。这与ES6模块化有很大差异(后面会介绍)。

```
// example.js
var x = 1
var addX = function (val) {
    x += val
    return x
}
module.exports = {
    x, addX
}

// index.js
var example = require('./example.js')
var test = require('./test.js')
console.log(example.addX(1)) // 2
console.log(example.x)       // 1
```

#### require 的内部处理流程

`require` 命令是 CommonJS 规范之中，用来加载其他模块的命令，它其实不是一个全局命令，而是指向当前模块的 `module.require` 命令，而后者又调用 Node 的内部命令 `Module._load`。

```
Module._load = function (request, parent, isMain) {
    // 1. 检查 Module._cache 是否缓存之中有指定模块
    // 2. 如果缓存之中没有，就创建一个新的 Module 实例
    // 3. 将它保存到缓存
    // 4. 使用 module.load() 加载指定的模块文件
    // 5. 如果加载/解析过程保存，就从缓存删除该模块
    // 6. 返回该模块的 module.exports
}
```

上面第四步，采用 `module.compile()` 执行指定模块的脚本，逻辑如下：

```
Module.prototype._compole = function (content, filename) {
    // 1. 生成一个 require 函数，指向 module.require
    // 2. 加载其他辅助方法到 require
    // 3. 将文件内容放到一个函数中，该函数可以调用 require
    // 4. 执行该函数
}
```

上面第一步和第二步，`require` 函数及其辅助方法主要如下：

```
> require()：加载外部模块
> require.resolve()：让模块名解析到一个绝对路径
> require.main：指向主模块
> require.cache：指向所有缓存的模块
> require.extensions：根据文件的后缀名，调用不同的执行函数
```

一旦 `require` 函数准备完毕，整个索要加载的脚本内容，就被放到一个新的函数中，这样可以避免污染全局环境中。该函数的参数包括 `require`、`module`、`exports`，以及其他一些参数。

```
(function (exports, require, module, __filename, __dirname) {
    // your code injected here
})
```

`Module._compile` 方法是同步执行的，所以`Module._load` 要等它执行完成，才会向用户返回 `module.exports` 的值。

## AMD（Asynchronous Module Definition）：异步模块定义


与 CommonJS 规范同步加载模块不一样，AMD 规范采用异步加载模块，模块的加载不影响后面语句的运行，所有依赖这个模块的语句都定义在回调函数中，等到模块加载完成后，回调函数才会执行。

由于 Node.js 主要用于服务器编程，模块文件已经在本地磁盘，所以加载起来比较快，不用考虑非同步加载的方式，所以 CommonJS 规范比较适用。但是，如果是浏览器环境，要从服务器端加载模块，这时就必须采用非同步模式，因此浏览器一般采用 AMD 规范。

由于 JavaScript 原生不支持，使用 AMD 规范开发需要用到对应的库函数，这里介绍 requirejs 库实现 AMD 规范模块化。

### 语法

* require.config：指定引用路径
* define：定义模块
* require：加载模块

```
// html 中引用 require.js 以及 main.js
<script src="js/require.js" data-main="js/main"></script>

// main.js 入口文件/主模块
require.config({
    baseUrl: 'js/lib',
    paths: {
        'jquery': 'jquery.min',     // 实际路径为 js/lib/jquery.min.js
        'underscore': 'underscore.min'
    }
})
// 执行基本操作
require(['jquery', 'underscore'], function ($, _) {
    // 处理逻辑
})
```

AMD 规范允许输出的模块兼容 CommonJS 规范，这时，`define` 方法需要写成这样：

```
define(function (require, exports, module) {
    var someModule = require('someModule')
    var anotherModule = require('anotherModule')

    someModule.doTheAwesome()
    anotherModule.doMoarAwesome()

    exports.asplode = function () {
        someModule.doTheAwesome()
        anotherModule.doMoarAwesome()
    }
})
```

# 参考文献

* [前端模块化：CommonJS,AMD,CMD,ES6](https://juejin.im/post/5aaa37c8f265da23945f365c)
* [深入 CommonJs 与 ES6 Module](https://segmentfault.com/a/1190000017878394)
* [JavaScript modules 模块 - MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Modules)
* [CommonJS规范 - 阮一峰](https://javascript.ruanyifeng.com/nodejs/module.html)
