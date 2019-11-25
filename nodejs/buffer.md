[返回Node.js文档](./README.md)

# 概述

在引入 [TypedArray](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) 之前，JavaScript 语言没有用于读取或操作二进制数据流的机制。`Buffer` 类是作为 Node.js API 的一部分引入的，用于在 TCP 流、文件系统操作以及其他上下文中与八位字节流进行交互。

现在可以使用 TypedArray，Buffer 类以更优化和更适合 Node.js 的方式实现了 Uint8Array API。

Buffer 类的实例类似于从 0 到 255 之间的整数数组（其他整数会通过 `& 255` 操作强制转换到此范围），但对应于 V8 堆外部的固定大小的原始内存分配。Buffer 的大小在创建时确定，且无法更改。

Buffer 类在全局作用域中，因此无需使用 `require('buffer').Buffer`。

```
// 创建一个长度为 10 且用 0 填充的 Buffer
const buf1 = Buffer.alloc(10)

// 创建一个长度为 10 且用 0×1 填充的 Buffer
const buf2 = Buffer.alloc(10, 1)

// 创建一个长度为 10 且未初始化的 Buffer。
// 这个方法比 Buffer.alloc() 更快，
// 但返回的 Buffer 实例可能包含旧数据，
// 因此需要用 fill() 或 write() 重写
const buf3 = Buffer.allocUnsafe(10)

// 创建一个包含 [0×1, 0×2, 0×3] 的 Buffer
const buf4 = Buffer.from([1, 2, 3])

// 创建一个包含 UTF-8 字节 [0×73, 0×c3, 0×a9, 0×73, 0×74]
const buf5 = Buffer.from('test')

// 创建一个包含 Latin-1 字节 [0x74, 0xe9, 0x73, 0x74] 的 Buffer。
const buf6 = Buffer.from('tést', 'latin1')
```

# Buffer API

## Buffer.from()、Buffer.alloc() 与 Buffer.allocUnsafe()

在 6.0.0 之前的 Node.js 版本中，Buffer 实例是用 Buffer 构建函数创建的，该函数根据提供的参数以不同方式分配返回的 Buffer：

* 将数字作为第一个参数传递给 `Buffer()`（例如 `new Buffer(10)`）会分配一个指定大小的新建的 `Buffer` 对象。在 Node.js 8.0.0 之前，为此类 `Buffer` 实例分配的内存是未初始化的，可能包含敏感数据。此类 Buffer 实例随后必须被初始化，可以使用 `buf.fill(0)` 或写满整个 `Buffer`。虽然这种行为是为了提高性能，但开发经验表明，创建一个快速但未初始化的 Buffer 与创建一个速度更慢但更安全的 Buffer 之前需要有更明确的区分。从 Node.js8.0.0 开始，`Buffer(num)` 和 `new Buffer(num)` 将返回已初始化内存的 Buffer。
* 传入字符串、数组或 Buffer 作为第一个参数，则会将窜入的对象的数据拷贝到 Buffer中。
* 传入 ArrayBuffer 或 SharedArrayBuffer，则返回一个与给定的数组 buffer 共享已分配内存的 Buffer。

由于 `new Buffer()` 的行为因第一个参数的类型而异，因此当未执行参数验证或 Buffer 初始化时，可能会无意中将安全性和可靠性问题引入应用程序。

例如，如果攻击者可以使应用程序接收到数字（实际期望的是字符串），则应用程序可能调用 new `Buffer(100)` 而不是 `new Buffer("100")`，它将分配一个 100 个字节的 buffer 而不是分配一个内容为 “100” 的 3 个字节的 buffer。 使用 JSON API 调用时，是非常有可能发生这种情况的。 由于 JSON 区分数字和字符串类型，因此它允许在简单的应用程序可能期望始终接收字符串的情况下注入数字。 在 Node.js 8.0.0 之前，100 个字节的 buffer 可能包含任意预先存在的内存数据，因此可能会用于向远程攻击者暴露内存中的机密。 从 Node.js 8.0.0 开始，由于数据会用零填充，因此不会发生内存暴露。 但是，其他攻击仍然存在，例如导致服务器分配非常大的 buffer，导致性能下降或内存耗尽崩溃。

为了使 Buffer 实例的创建更可靠且更不容易出错，各种形式的 new Buffer() 构造函数都已被弃用，且改为单独的 `Buffer.from()`，`Buffer.alloc()` 和 `Buffer.allocUnsafe()` 方法。

开发者应将 `new Buffer()` 构造函数的所有现有用法迁移到这些新的 API。

* `Buffer.from(array)`：返回一个新 Buffer，其中包含提供的八位字节数组的副本。
* `Buffer.from(arrayBuffer[, byteOffeset [, length]])`：返回一个新的 Buffer，它与给定的 ArrayBuffer 共享相同的已分配内存。
* `Buffer.from(buffer)`：返回一个新的 `buffer`，其中包含给定 `Buffer` 的内容的副本。
* `Buffer.from(string[, encoding])`：返回一个新的 `Buffer`，其中包含提供的字符串的副本。
* `Buffer.alloc(size[, fill[, encoding]])`：返回一个指定大小的新建的已初始化的 `Buffer`。此方法比 `Buffer.allocUnsafe(size)` 慢，但能确保新创建的 `Buffer` 实例永远不会包含可能敏感的旧数据。如果 `size` 不是数字，则将会抛出 `TypeError`。
* `Buffer.allocUnsafe(size)` 和 `Buffer.allocUnsafeSlow(size)` ：返回一个指定大小的新建未初始化的 `Buffer`。由于 `Buffer` 是未初始化的，因此分配的内存片段可能包含敏感的旧数据。

如果 `size` 小于或等于 `Buffer.poolSize` 的一半，则 `Buffer.allocUnsafe()` 返回的 `Buffer` 实例可能是从共享的内部内存池中分配。`Buffer.allocUnsafeSlow()` 返回的实例则从不使用共享的内部内存池。

### --zero-fill-buffers 命令行选项

新增与：v5.10.0

`--zero-fill-buffers` 命令选项启动 Node.js，以使所有新分配的 `Buffer` 实例在创建时默认使用零来填充。在 Node.js8.0.0 之前，这包括 `new Buffer(size)`、`Buffer.allocUnsafe()`、`Buffer.allocUnsafeSlow()` 和 `new SlowBuffer(size)` 分配的 buffer。 从 Node.js 8.0.0 开始，无论是否使用此选项，使用 new 分配的 buffer 始终用零填充。 使用这个选项可能会对性能产生重大的负面影响。 建议仅在需要强制新分配的 Buffer 实例不能包含可能敏感的旧数据时，才使用 --zero-fill-buffers 选项。

```
$ node --zero-fill-buffers
> Buffer.allocUnsafe(5)
<Buffer 00 00 00 00 00>
```

### Buffer.allocUnsafe() 与 Buffer.allocUnsafeSlow() 不安全的原因

当调用 `Buffer.allocUnsafe()` 与 `Buffer.allocUnsafeSlow()` 时，分配的内存片段是未初始化的（没有被清零）。虽然这样的设计使得内存的分配非常快，但分配的内存可能包含敏感的旧数据。使用由 `Buffer.allocUnsafe()` 创建的 `Buffer` 没有完全重写内存，当读取 Buffer 内存时，可能使旧数据泄漏。

虽然使用 `Buffer.allocUnsafe()` 有明显的性能优势，但必须格外小心，以避免安全泄漏引入应用程序。

## Buffer 与字符编码

当字符串数据被存储入 `Buffer` 实例或从 `Buffer` 实例中被提取时，可以指定一个字符编码。

```
const buf = Buffer.from('hello world', 'ascii')

console.log(buf.toString('hex'))    // 68656c6c6f20776f726c64
console.log(buf.toString('base64')) // aGVsbG8gd29ybGQ=

console.log(Buffer.from('fhqwhgads', 'ascii'))
// <Buffer 66 68 71 77 68 67 61 64 73>

console.log(Buffer.from('fhqwhgads', 'utf16le'))
//<Buffer 66 00 68 00 71 00 77 00 68 00 67 00 61 00 64 00 73 00>
```

Node.js 当前支持的字符编码有：

* `'ascii'`：仅适用于 7 位 ASCII 数据。此编码速度很快，如果设置则会剥离高位。
* `'utf8'`：多字节编码的 Unicode 字符。许多网页和其他文档格式都使用 UTF-8。
* `'utf16le'`：2 或 4 个字节，小端序编码的 Unicode 字符。支持代理对（U+10000 至 U+10FFFF）。
* `'ucs2'`：`'utf16le'` 别名。
* `'base64'`：Base64 编码。当从字符串创建 Buffer 时，此编码也会正确地接受 [RFC 4648 第 5 节](http://nodejs.cn/s/j8aS4R) 中指定的 “URL 和文件名安全字母”。
* `'latin1'`：一种将 Buffer 编码成单字节编码字符串的方法。
* `'binary'`：`'latin1'` 的别名。
* `'hex'`：将每个字节编码成功两个十六进制的字符。

现代的 Web 浏览器遵循 [WHATWG 编码标准](http://nodejs.cn/s/sasgQF)，将 `'latin1'` 和 `ISO-8859-1'` 别名为 `'win-1252'`。这意味着当执行 `http.get()` 之类的操作时，如果返回的字符集是 WHATWG 规范中列出的字符集之一，则服务器可能实际返回 `'win-1252'` 编码的数据，而使用 `'latin1'` 编码可能错误地解码字符。

## Buffer 与 TypedArray

Buffer 实例也是 [Unit8Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array) 实例，但是与 [TypedArray](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) 有微小的不同。例如 `Array.Buffer#slice()` 会创建切片的拷贝，而 `Buffer#slice()` 是在现有的 Buffer 上创建而不拷贝，这使得 ``Buffer#slice()` 效率更高。

也可以从一个 Buffer 创建新的 (https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) 实例，但需要注意以下事项：

1. Buffer 对象的内存是被拷贝到 TypedArray，而不是共享。
2. Buffer 对象的内存是被解释为不同元素的数组，而不是目标类型的字节数组。也就是说 `new Unit32Array(Buffer.from([1, 2, 3, 4]))` 会创建一个带有 4 个元素 `[1, 2, 3, 4]` 的 `Uint32Array`，而不是带有单个元素 `[0x1020304]` 或 `[0x4030201]` 的 `Uint32Array`。

通过使用 `TypedArray` 对象的 `.buffer` 属性，可以创建一个与 TypedArray 实例共享相同内存的新 `Buffer`。

```
const arr = new Uint16Array(2)

arr[0] = 5000
arr[1] = 4000

// 拷贝 `arr` 的内存
const buf1 = Buffer.from(arr)
// 与 `arr` 共享内存
const buf2 = Buffer.from(arr.buffer)

console.log(buf1)   // <Buffer 88 a0>
console.log(buf2)   // <Buffer 88 13 a0 0f>

arr[1] = 6000
console.log(buf1)   // <Buffer 88 a0>
console.log(buf2)   // <Buffer 88 13 70 17>
```

当使用 TypedArray 的 `.buffer` 创建 Buffer 时，也可以通过传入 `byteIffset` 和 length 参数只使用 TypedArray 的一部分。

```
const arr = new Uint16Array(20)
const buf = Buffer.from(arr.buffer, 0, 16)

console.log(buf.length)     // 16
```

`Buffer.from` 与 `TypedArray.from` 有着不同的实现。具体来说，TypedArray 可以接受第二个参数作为映射函数，在类型数组的每个元素上调用：

```
TypedArray.from(source[, mapFn[, thisArg]])
```

`Buffer.from()` 则不支持映射函数的使用：

* Buffer.from(array)
* Buffer.from(buffer)
* Buffer.from(arrayBufer[, byteOffset[, length]])
* Buffer.from(string[, encoding])

## Buffer 与迭代器

Buffer 实例可以使用 `for...of` 语法进行迭代：

```
const buf = Buffer.from([1, 2, 3])

for (let b of buf) {
    console.log(b)
}
// 1
// 2
// 3
```

## Buffer 类

Buffer 类是一个全局变量，用于直接处理二进制数据。它可以使用多种方式构建。

* `new Buffer(buffer)`：已弃用，改为 `Buffer.from(buffer)`。
* `new Buffer(size)`：已弃用，改为 `Buffer.alloc()` 或 `Buffer.allocUnsafe()`。
* `new Buffer(string[, encoding])`：已弃用，改为 `Buffer.from(string[, encoding])`。

### Buffer.alloc(size[, fill[, encoding]])

* size：<integer> 新 Buffer 的所需长度。
* fill：<string> | <Buffer> | <Uint8Array> | <integer> 用于填充新 Buffer 的值。默认：`0`。
* encoding：<string> 如果 fill 是一个字符串，则这是它的字符编码。默认值：`'utf8'`。

```
const buf = Buffer.alloc(5)

console.log(buf)
// <Buffer 00 00 00 00 00>
```

如果 `size` 大于 [buffer.constant.MAX_LENGTH](http://nodejs.cn/s/aBiAe5) 或小于 0，则抛出 [ERR_INVALID_OPT_VALUE](http://nodejs.cn/s/ouMFyk)。如果 size 为 0，则创建一个零长度的 Buffer。

如果指定了 fill，则分配的 Buffer 通过调用 `buf.fill(fill)` 进行初始化。

```
const buf = Buffer.alloc(5, 'a')

console.log(buf)
// <Buffer 61 61 61 61 61>
```

如果同时指定了 fill 和 encoding，则分配的 Buffer 通过调用 `buf.fill(fill, encoding) 进行初始化。

```
const buf = Buffer.alloc(11, 'aGVsbG8gd29ybGQ=', 'base64')

console.log(buf)
// <Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
```

调用 Buffer.alloc() 可能比替代的 Buffer.allocUnsafe() 慢得多，但能确保新创建的 Buffer 实例的内容永远不会包含敏感的数据。

如果 size 不是一个数字，则抛出 TypeError。


### Buffer.allocUnsafe(size)

* size：<integer> 新建的 Buffer 长度。

创建一个大小为 size 字节的新 Buffer。如果 size 大于 [buffer.constants.MAX_LENGTH](http://nodejs.cn/s/aBiAe5) 或小于 0，则抛出 [ERR_INVALID_OPT_VALUE](http://nodejs.cn/s/ouMFyk)。如果 size 为 0，则创建一个零长度的 Buffer。

以这种方式创建的 Buffer 实力的底层内存是未初始化的。新创建的 Buffer 内容是未知的，可能包含敏感数据。使用 `Buffer.alloc()` 可以创建以零初始化的 Buffer 实例。

```
const buf = Buffer.allocUnsafe(10)

console.log(buf)
// (打印内容可能有所不同)<Buffer 0e 00 00 00 06 02 00 00 06 00>

buf.fill(0)

console.log(buf)
// <Buffer 00 00 00 00 00 00 00 00 00 00>
```

如果 size 不是一个数字，则抛出 TypeError。

Buffer 模块会预分配一个内部的大小为 [Buffer.poolSize](http://nodejs.cn/s/dZo4K3) 的 Buffer 实例，作为快速分配的内存池，用于使用 `Buffer.allocUnsafe()` 创建的 Buffer 实例、或废弃的 `new Buffer(size)` 构造器仅当 size 小于或等于 `Buffer.poolSize >> 1` (Buffer.poolSize 除以2再向下取整)。

`Buffer.alloc(size, fill)` 和 `Buffer.allocUnsafe(size).fill(fill)` 区别： 

*  `Buffer.alloc(size, fill)`：永远不会使用 Buffer 池。
*  `Buffer.allocUnsafe(size).fill(fill)`：在 size 小于等于 `Buffer.poolSize` 的一半时将会使用内部的 Buffer 池。

这个差异虽然很微妙，但当应用程序需要 `Buffer.allocUnsafe()` 提供额外性能时，则非常重要。


### Buffer.allocUnsafeSlow(size)

* size：<integer> 新建的 Buffer 长度。

创建一个大小为 size 字节的新 Buffer。如果 size 大于 [buffer.constants.MAX_LENGTH](http://nodejs.cn/s/aBiAe5) 或小于 0，则抛出 [ERR_INVALID_OPT_VALUE](http://nodejs.cn/s/ouMFyk)。如果 size 为 0，则创建一个零长度的 Buffer。

以这种方式创建的 Buffer 实力的底层内存是未初始化的。新创建的 Buffer 内容是未知的，可能包含敏感数据。使用 `buf.fill(0)` 可以创建以零初始化的 Buffer 实例。

当使用 `Buffer.allocUnsafe()` 创建新的 Buffer 实例时，如果要分配的内存小于 4KB ，则会从一个预分配的 Buffer 切割出来。这样可以避免垃圾回收机制因创建太多独立 Buffer 而过度使用。这种方式通过消除跟踪和清理的需要来改进性能和内存使用。

当开发人员需要在内存池中保留一小块内存时，可以使用 `Buffer.allocUnsafeSlow()` 创建一个非内存池的 Buffer 实例并拷贝相关的比特位来。

```
// 需要保留一小块内存
const store = []

socket.on('readable', () => {
    let data
    while (null !== (data = readable.read())) {
        // 为剩下的数据分配内存
        const sb = Buffer.allocUnsafeSlow(10)

        // 拷贝数据到新分配的内存
        data.copy(sb, 0, 10)

        store.push(data)
    }
})
```

`Buffer.allocUnsafeSlow()` 应该只用作开发人员已经在其应用程序中观察到过度的内存之后的最后手段。

如果 size 不是一个数字，则抛出 TypeError。

### Buffer.byteLength(string[, encoding])

* string：<string> | <Buffer> | <TypedArray> | <DataView> | <ArrayBuffer> | <SharedArrayBuffer> 要计算长度的值。
* encoding：<string> 如果 string 是字符串，则这是它的字符编码。默认值 `'utf8'`。  

### Buffer.compare(buf1, buf2)

### Buffer.concat(list[, totalLength])

### Buffer.from(array)

### Buffer.from(arrayBuffer[, byteOffset[, length]])

### Buffer.from(buffer)

### Buffer.from(object[, offsetOrEncoding[, length]])

### Buffer.from(string[, encoding])

### Buffer.isBuffer(obj)

### Buffer.isEncoding(encoding)

### Buffer.poolSize

### buf[index]

# 参考文献

* [Buffer（缓冲器）](http://nodejs.cn/api/buffer.html)