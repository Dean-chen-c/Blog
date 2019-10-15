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

> 首先原始类型存储的都是**值**，是没有函数可以调用的
> number 可以表示 2^53 Bigint 可以表示**任意大**的数
> 对象类型存储的是**地址**（指针）

### typeof

一般用于判断值是不是**undefined**

typeof 对于**原始类型**来说，除了 **null** 都可以显示正确的类型

```javascript
typeof 1; // 'number'
typeof "1"; // 'string'
typeof undefined; // 'undefined'
typeof true; // 'boolean'
typeof Symbol(); // 'symbol'
typeof null; // 'object'
```

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

> **Symbol.hasInstance** 用于判断某对象是否为某构造器的实例。 因此你可以用它自定义 instanceof 操作符在某个类上的行为。

```javascript
class MyArray {
  static [Symbol.hasInstance](instance) {
    return Array.isArray(instance);
  }
}
console.log([] instanceof MyArray); // true

class MyString {
  static [Symbol.hasInstance](instance) {
    return typeof instance === "string";
  }
}
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

### 类型转换

在 JS 中类型转换只有三种情况

- 转换为布尔值
- 转换为数字
- 转换为字符串

#### 转 Boolean

转换为 false：

- undefined
- null
- fasle
- ''
- 0
- -0
- NaN

#### 转 String

toString()

#### 转 Number

Number()

#### 对象转原始类型

- 对象在转换类型的时候，会调用内置的\[[ToPrimitive]] 函数
- 如果已经是原始类型了，那就不需要转换了
- 调用 x.valueOf()，如果转换为基础类型，就返回转换的值
- 调用 x.toString()，如果转换为基础类型，就返回转换的值

```javascript
let a = {
  valueOf() {
    return 0;
  },
  toString() {
    return "1";
  },
  [Symbol.toPrimitive]() {
    return 2;
  }
};
1 + a; // => 3

"a" + +"b"; // -> "aNaN"
```
