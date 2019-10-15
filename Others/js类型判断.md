# JavaScript 类型判断

### JS 的数据类型

js 数据类型如下：

- 7 种原始类型
  - Number
  - String
  - Boolean
  - null
  - undefine
  - BigInt
  - Symbol
- Object 类型

>首先原始类型存储的都是值，是没有函数可以调用的
>number可以表示2^53 Bigint可以表示任意大的数

### typeof

一般用于判断值是不是**undefined**

### instanceof

从英文可以直面翻译：什么什么的实例
所以这个方法跟原型链有关。

> **instanceof** 运算符用于检测构造函数的 prototype 属性是否出现在某个实例对象的原型链上。

```javascript
new Number(1) instanceof Object;
true;
new Number(1) instanceof Number;
true;
```

### Object.prototype.toString.call()

```javascript
function type(obj) {
  return Object.prototype.toString
    .call(obj)
    .replace(/\[object\s(\w+)\]$/, "$1")
    .toLowerCase();
}
```

### null 类型判断

直接用强等于 **===**

### 判断数组

Array.isArray()
