[返回Node.js文档](./README.md)

# 1. 简介

Node 是 JavaScript 的服务器端运行环境。

所谓运行环境有两层含义：

* JavaScript 通过 Node 在服务器上运行，这个意义上，Node 类似于 JavaScript 虚拟机。
* Node 提供工具库，使得 JavaScript 语言与操作系统互动（比如读写文件、新建子进程），这个意义上，Node 又是 JavaScript 的工具库。

## 1.1. 安装与更新

访问官方网站 [nodejs.org](nodejs.org) 或者 [github.com/nodesource/distributions](github.com/nodesource/distributions)，查看Node的最新版本和安装方法。

安装完成后，运行一下命令，查看是否安装成功

```
$ node --version
# 或者
$ node -v
```

## 1.2. 版本管理工具 nvm

如果一台机器需要安装多个版本的 node.js，就需要用到版本管理工具 nvm。

安装可以参考：https://github.com/nvm-sh/nvm

```
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.1/install.sh | bash
# 或者
$ wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.1/install.sh | bash
```

再配置 `profile`  (`~/.bash_profile`, `~/.zshrc`, `~/.profile`, or `~/.bashrc`)

```
# 安装最新版本
$ nvm install node

# 安装指定版本
$ nvm install 0.12.1

# 使用已安装的最新版本
$ nvm use node

# 使用指定版本的node
$ nvm use 0.12

# 本地已安装的版本列表以及正在使用的版本
$ nvm ls

# 查看远程可以安装的版本号
$ nvm ls-remote
```

## 1.3. 基本用法

### 1.3.1. 运行 JavaScript 文件

```
$ node demo
# 或者
$ node demo.js
```

### 1.3.2. 执行代码字符串

```
$ node -e 'console.log("Hello World")'
Hello World
```

### 1.3.3. REPL 环境

在命令行键入node命令，后面没有文件名，就进入一个Node.js的REPL环境（Read–eval–print loop，”读取-求值-输出”循环），可以直接运行各种JavaScript命令。

```
$ node
> console.log('Hello World')
Hello World
>
```

如果使用参数 –use_strict，则REPL将在严格模式下运行。

REPL是Node.js与用户互动的shell，各种基本的shell功能都可以在里面使用，比如使用上下方向键遍历曾经使用过的命令。

特殊变量下划线（_）表示上一个命令的返回结果

```
$ node
> 1 + 1
2
> _ + 1
3
```

在REPL中，如果运行一个表达式，会直接在命令行返回结果。如果运行一条语句，就不会有任何输出，因为语句没有返回值。

```
> x = 1
1
> var x = 1
```

上面代码的第二条命令，没有显示任何结果。因为这是一条语句，不是表达式，所以没有返回值。

## 1.4. 异步操作

Node 采用 V8 引擎处理 JavaScript 脚本，最大的特点就是单线程运行。一次只能执行一个任务。这导致Node大量采用异步操作（asynchronous operation），即任务不是马上执行，而是插在任务队列的尾部，等到前面的任务运行完后再执行。

由于这种特性，某一个任务的后续操作，往往采用回调函数（callback）的形式进行定义。

```
var isTrue = function (value, callback) {
    if (value === true) {
        callback(null, 'Value was true.')
    } else {
        callback(new Error('Value was false.'))
    }
}
```

Node 约定：如果某个函数需要回调函数作为参数，则回调函数是最后一个参数。另外，回调函数本身的第一个参数，约定为上一步传入的错误对象。

```
var callback = function (error, value) {
    if (error) {
        return console.log(error)
    }
    console.log(value)
}
```

上面代码中，callback 第一个参数是 Error 对象，第二个参数才是真正的数据参数。这是因为回调函数主要用于异步操作，当回调函数运行时，前期的操作早结束了，错误的执行栈早就不存在了，传统的错误捕捉机制 try...catch 对于异步操作行不通，所以只能把错误交给回调函数处理。

```
try {
    db.User.get(userId, function (err, user) {
        if (err) {
            throw err
        }
        // ...
    })
} catch (e) {
    console.log('oh, no!')
}
```

上面代码中，`db.User.get` 是一个异步操作，等到抛出错误时，可能所在的 try...catch 代码块早就执行结束了，这会导致错误无法捕获。所以，Node 统一规定，一旦异步操作发生错误就把错误对象传递到回调函数。

如果没发生错误，回调函数的第一个参数就传入 null。这种写法有一个很大的好处，就是说只要判断回调函数的第一个参数，就知道有没有出错，如果不是 null，肯定出错了。另外这样还可以层层传递错误。

```
if (err) {
    // 除了放过No Permission错误意外，其他错误传给下一个回调函数
    if (!err.noPermission) {
        return next(err)
    }
}
```

## 1.5. 全局对象和全局变量

Node 提供以下几个全局对象，它们是所有模块都可以调用的：

* `global`：表示 Node 所在的全局环境，类似于浏览器的 `window` 对象。需要注意的是，如果在浏览器中生命一个全局变量，实际上是声明了一个全局对象的属性，比如 `var x = 1` 等同于设置 `window.x = 1`。但是 Node不是这样，至少在模块中不是这样（REPL环境的行为与浏览器一致）。在模块声明中，声明 `var x = 1`，该变量不是 `global` 对象的属性，`global.x` 等于 `undefined`。这是因为模块的全局变量都是该模块私有的，其他模块无法获取到。
* `process`：该对象表示 Node 所处的当前进程，允许开发者与该进程互动。
* `console`：指向 Node 内置的 console 模块，提供命令行环境中的标准输入、标准输出的功能。

Node 还提供一些全局函数：

* `setTimeout()`：用于在指定毫秒之后，运行回调函数。实际的调用间隔，还取决于系统因素。间隔的毫秒数在 1ms 到 2,147,483,647ms (约24.8天)之间。如果超过这个范围，会被自动改为 1ms。该方法返回一个整数，代表这个新建定时器的编号。
* `clearTimeout()`：用于终止一个 `setTimeout` 方法新建的定时器。
* `setInterval()`：用于每隔一定毫秒调用回调函数。由于系统因素，可能无法保证每次调用之间正好间隔指定的毫秒数，但只会多于这个间隔，而不会少于它。指定的毫秒数必须是 1ms 到 2,147,483,647ms (约24.8天)之间的整数，如果超过这个范围，会被自动改为 1ms。该方法返回一个整数，代表这个新建定时器的编号。
* `clearInterval()`：终止一个用 `setInterval` 方法新建的定时器。
* `require`：用于加载模块。
* `Buffer()`：用于操作二进制数据。

Node 提供两个全局变量，都以两个下划线开头。

* `__filename`：指向当前运行的脚本文件名。
* `__dirname`：指定当前运行脚本所在的目录。

除此之外，还有一些对象实际上是模块内部的全局变量，指向的对象根据模块不同而不同，但是所有模块都适用，可以看做是伪全局变量，主要为 module，module,exports，exports 等。

# 2. 模块化结构

## 2.1. 概述

Node.js 采用模块化结果，按照 [CommonJS 规范](http://wiki.commonjs.org/wiki/CommonJS)定义和适用模块。模块与文件是一一对应关系，即加载一个模块，实际上就是加载对应的一个模块文件。

require 命令用于指定加载模块，加载时可以省略脚本文件的后缀名。

```
var circle = require('./circle.js')
// 或者
var circle = require('./circle')
```

require 方法的参数是模块文件的名字。它分成两种情况，第一种情况是参数种含有文件路径（比如上例），这时路径是相对于当前脚本所在的目录；第二种情况是参数中不带有文件路径，这时 Node 到模块的安装目录，去寻找已安装的模块：

```
var bar = require('bar')
```

有时候，一个模块本身就是一个目录，目录中包含多个文件。这个时候，Node 在 package.json 文件中，寻找 main 属性所指明的模块入口文件。

```
{
    "name": "bar",
    "main": "./lib/bar.js"
}
```

上面代码中，模块的启动文件为 lib 目录下的 bar.js。当使用 `require('bar')` 命令加载该模块时，实际上加载的是 `./node_modules/bar/lib/bar.js` 文件。下面的写法会起到同样的效果。

```
var bar = require('./node_modules/bar/lib/bar.js')
```

如果模块目录没有 package.json 文件，Node.js 会尝试在模块目录中寻找 index.js 或 index.node 文件进行加载。

模块一旦被加载后，就会被系统缓存。如果第二次还加载该模块，则会返回缓存中的版本，这意味着模块实际上会执行一次。如果希望模块执行多次，则可以让模块返回一个函数，然后多次调用该函数。

## 2.2. 核心模块

如果只是在服务器运行 JavaScript 代码，用处并不大，因为服务器脚本语言已经很多种了。Node.js 的用于在于，它本身还提供了一系列功能模块，与操作系统互动。这些核心的功能模块，不用安装就可以使用，下面是它的清单。

* http：提供 HTTP 服务器功能
* url：解析 URL
* fs：与文件系统交互
* querystring：解析 URL 的查询字符串
* child_process：新建子进程
* util：提供一系列实用小工具
* path：处理文件路径
* crypto：提供加密和解密功能，基本上是对 OpenSSL 的包装

上面这些核心模块，源码都在 Node 的 lib 子目录中。为了提高运行速度，他们会安装时都会被编译成二进制文件。

核心模块总是最优先加载的。如果你自己写了一个 HTTP 模块， `require('http')` 加载的还是核心模块。

## 2.3. 自定义模块

Node 模块采用 CommonJS 规范。只要符合这个规范就可以自定义模块。
下面是一个最简单的模块，假定新建一个 foo.js 文件，写入以下内容。

```
// foo.js
module.exports = function (x) {
    console.log(x)
}
```

上面代码就是一个模块，它通过 module.exports 变量，对外输出一个方法。

这个模块的使用方法如下：

```
// index.js

var m = require('./foo')
m('这是自定义模块')
```

上面代码通过require命令加载模块文件foo.js（后缀名省略），将模块的对外接口输出到变量m，然后调用m。这时，在命令行下运行index.js，屏幕上就会输出“这是自定义模块”。

```
$ node index
这是自定义模块
```

module 变量是整个模块文件的顶层变量，它的 exports 属性就是模块向外输出的接口。如果直接输出一个函数（就像上面的 foo.js），那么调用模块就是调用一个函数。但是，模块也可以输出一个对象。下面对 foo.js 进行改写。

```
// foo.js
var out = new Object()
function p(string) {
    console.log(string)
}
out.print = p

module.exports = out
```

上面的代码表示模块输出out对象，该对象有一个print属性，指向一个函数。下面是这个模块的使用方法。

```
// index.js
var m = require('./foo')

m.print('这是自定义模块')
```

上面代码表示，由于具体的方法定义在模块的print属性上，所以必须显式调用print属性。

# 3. 异常处理

Node 是单线程运行环境，一旦抛出异常没有被捕获，就会引起整个进程的崩溃。所以，Node 的异常处理对于保证系统的稳定运行非常重要。

一般来说，Node 有三种方法传播一个错误：

* throw 语句抛出一个错误对象，即抛出异常
* 将错误对象传递给回调函数，由回调函数负责发出错误
* 通过 EventEmitter 接口，发出一个 error 事件

## 3.1. try...catch 结构

最常用的捕获异常的方式，就是使用try…catch结构。但是，这个结构无法捕获异步运行的代码抛出的异常。

```
try {
    process.nextTick(function () {
        throw new Error('error')
    })
} catch (e) {
    // cannot catch it
    console.log(e)
}

try {
    setTimeout(function(){
      throw new Error("error");
    },1)
} catch (err) {
    //can not catch it
    console.log(err);
}
```

上面代码分别用process.nextTick和setTimeout方法，在下一轮事件循环抛出两个异常，代表异步操作抛出的错误。它们都无法被catch代码块捕获，因为catch代码块所在的那部分已经运行结束了。

一种解决方法是将错误捕获代码，也放到异步执行。

```
function async (cb, err) {
    setTimeout(function () {
        try {
            if (true) {
                throw new Error('error')
            } else {
                cb('done')
            }
        } catch (e) {
            err(e)
        }
    }, 200)
}

async(function (res) {
    console.log('received: ', res)
}, function (err) {
    console.log('Error: async throw an exception: ', err)
})
// Error: async throw an exception:  Error: error
```

上面代码中，async函数异步抛出的错误，可以同样部署在异步的catch代码块捕获。

这两种处理方法都不太理想。一般来说，Node只在很少场合才用try/catch语句，比如使用JSON.parse解析JSON文本。

## 3.2. 回调函数

Node 采用的方法，是将错误对象作为第一个参数，传入回调函数。这样就避免了捕获代码与发生错误的代码不在同一个时间段的问题。

```
var fs = require('fs')
fs.readFile('foo.txt', function (err, data) {
    if (err !== null) {
        throw err
    }
    console.log(data)
})
```

上面代码表示，读取文件 `foo.txt` 是一个异步操作，它的回调函数有两个参数，第一个是错误对象，第二个是读取到的文件数据。如果第一个参数不是 null，就意味着发生错误，后面代码也就不再执行了。

下面是一个完整的例子。

```
function async2(continuation) {
  setTimeout(function() {
    try {
      var res = 42;
      if (true)
        throw new Error("woops!");
      else
        continuation(null, res); // pass 'null' for error
    } catch(e) {
      continuation(e, null);
    }
  }, 2000);
}

async2(function(err, res) {
  if (err)
    console.log("Error: (cps) failed:", err);
  else
    console.log("(cps) received:", res);
});
// Error: (cps) failed: woops!
```

## 3.3. EventEmitter 接口的 error 事件

发生错误的时候，也可以用 EventEmitter 接口抛出 error 事件。

```
var EventEmitter = require('events').EventEmitter
var emitter = new EventEmitter()

emitter.emit('error', new Error('something bad happened'))
```

使用上面的代码必须小心，因为如果没有对 error 事件部署监听函数，会导致整个应用程序崩溃。所以一般总是必须同步部署以下代码。

```
emitter.on('error', function (err) {
    console.log('出错：', err.message)
})
```

## 3.4. uncaughtException 事件

当一个异常未被捕获，就会触发 uncaughtException 事件，可以对这个事件注册回调函数，从而捕获异常。

```
var logger = require('tracer').console()

process.on('uncaughtException', function (err) {
    console.error('Error caught in uncaughtException event:', err)
})

try {
    setTimeout(function () {
        throw new Error('error')
    }, 1)
} catch (e) {
    // cannot catch it
    console.log(err)
}
```

只要给 uncaughtException 配置了回调，Node 进程不会异常退出，但异常发生的上下文已经丢失，无法给出异常发生的详细信息。而且，异常可能导致 Node 不能正常进行内存回收，出现内存泄漏。所以，当 uncaughtException 触发后，最好记录错误日志，然后结束 Node 进程。

```
process.on('uncaughtException', function (err) {
    logger.log(err)
    process.exit(1)
})
```

## 3.5. unhandledRejection 时间

iojs 有一个 unhandledRejection 事件，用来监听没有捕获的 Promise 对象的 rejected 状态。

```
var promise = new Promise(function (resolve, reject) {
    reject(new Error('Broken'))
})
promise.then(function (res) {
    console.log(res)
})
```

上面代码，promise 状态为 rejected，并且抛出一个错误。但是不会有任何反应，因为没有设置任何处理函数。

只要监听 unhandledRejection 事件，就能解决这个问题。

```
process.on('unhandledRejection', function (err) {
    console.log(err.stack)
})
```

需要注意的是， unhandledRejection 事件的监听函数有两个参数，第一个是错误对象，第二个是产生错误的 promise 对象。这里可以提高很多有用的信息。

```
var http = require('http')

http.createServer(function (req, res) {
    var promise = new Promise(function (resolve, reject) {
        reject(new Error('Broken'))
    })
    promise.info = { url: req.url }
}).listen(8000)

process.on('unhandledRejection', function (err, p) {
    if (p.info && p.info.url) {
        console.log('Error in URL', p.info.url)
    }
    console.error(err.stack)
})
```

上面代码会在出错时，输出用户请求的网址。

```
Error in URL /123
Error: Broken
    at /Users/systrive/code/article-fe/test/index.js:5:16
    at new Promise (<anonymous>)
    at Server.<anonymous> (/Users/systrive/code/article-fe/test/index.js:4:19)
    at emitTwo (events.js:126:13)
    at Server.emit (events.js:214:7)
    at parserOnIncoming (_http_server.js:619:12)
    at HTTPParser.parserOnHeadersComplete (_http_common.js:112:17)
```

## 3.6. 命令行脚本

Node 脚本可以作为命令行脚本使用

```
$ node foo.js
```

foo.js 文件的第一行，如果加入了解释器的位置，就可以将其作为命令行工具直接调用。

```
# !/usr/bin/env node
```

调用前，需要更改文件的执行权限

```
$ chmod u+x foo.js
$ ./foo.js arg1 arg2 ...
```

作为命令行脚本时，`console.log` 用于输出内容到标准输出，`process.stdin` 用户读取标准输入，`child_process.exec()` 用于执行一个 shell 命令。

# 4. 参考文献

* [Node.js 概述 | JavaScript 标准参考教程(alpha)](http://javascript.ruanyifeng.com/nodejs/basic.html)