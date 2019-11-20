[返回首页](../../README.md)

# instanceof 实现原理

> `instanceof` 运算符用于检测构造函数的 prototype 属性是否出现在某个实例对象的原型链上

#### 语法：
```
object instanceof constructor
```

#### 参数

* `object`：某个实例对象
* `constructor`：某个构造函数

#### 返回值

* true/false

#### 例子
```
function C(){} 
function D(){} 

var o = new C();

o instanceof C; // true，因为 Object.getPrototypeOf(o) === C.prototype
o instanceof D; // false，因为 D.prototype 不在 o 的原型链上

o instanceof Object; // true，因为 Object.prototype.isPrototypeOf(o) 返回 true
C.prototype instanceof Object // true，同上
```

### instanceof 实现代码

思路：A instanceof B, 在A的原型链中层层查找，是否有原型等于B.prototype，如果一直找到A的原型链的顶端(null;即Object.proptotype.__proto__),仍然不等于B.prototype，那么返回false，否则返回true.

```
function instance_of (L, R) {
    let O = R.prototype
    L = L.__proto__

    while (true) {
        if (L === null) {
            return false
        }
        if (L === O) {
            return true
        }
        L = L.__proto__
    }
}
```


# 参考文献

* [instanceof - MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof)
* [ instanceof 的实现原理是什么](https://juejin.im/post/5cab0c45f265da2513734390#heading-1)