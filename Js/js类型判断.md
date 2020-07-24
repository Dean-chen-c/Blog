# JavaScript 类型判断

### JS 的数据类型

js 数据类型如下：

- 7 种原始类型(基本)
  - Number
  - String
  - Boolean
  - null
  - undefine
  - BigInt
  - Symbol
- Object 类型（引用）

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
typeof console.log; //function
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

#### instanceof 能否判断基本数据类型？

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

#### 手动实现 instanceof

```javascript
function myInstanceof(left, right) {
  if (typeof left !== "object" || left === null) return false;
  const rightPrototype = right.prototype;
  let proto = left.__proto__; //Object.getPrototypeOf()
  while (true) {
    if (proto === null) return false;
    if (proto === rightPrototype) return true;
    proto = left.__proto__;
  }
}
```

### Object.prototype.toString.call()

```javascript
function type(obj) {
  return Object.prototype.toString
    .call(obj)
    .replace(/^\[object\s(\w+)\]$/, "$1")
    .toLowerCase();
}
function type(obj){
return Object.prototype.toString.call(obj).split(' ')[1].slice(0,-1).toLowerCase()
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

| 原始值    | 转换目标 | 结果                  |
| --------- | :------: | --------------------- |
| number    | Boolean  | 除 0 -0 NaN 都为 true |
| string    | Boolean  | 除''都为 true         |
| undefined | Boolean  | false                 |
| null      | Boolean  | false                 |
| Object    | Boolean  | true                  |

#### 转 String

显式：toString() String()

| 原始值           | 转换目标 | 结果              |
| ---------------- | :------: | ----------------- |
| number           |  String  | '1'               |
| boolean function |  String  | 'function(){}'    |
| array            |  String  | '' '1,2,3'        |
| null,undefined   |  String  | 'null undefined'  |
| Object           |  String  | '[object Object]' |

#### 转 Number

显式：Number()

| 原始值    | 转换目标 | 结果        |
| --------- | :------: | ----------- |
| string    |  String  | 'a'-->'NaN' |
| object    |  String  | NaN         |
| array     |  String  | [][1] ['a'] |
| null      |  String  | 0           |
| undefined |  String  | NaN         |

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

如何让 if(a == 1 && a == 2)条件成立

```javascript
var a = {
  value: 0,
  valueOf: function() {
    this.value++;
    return this.value;
  }
};
console.log(a == 1 && a == 2);
```

### 隐式强制类型转换

```javascript
var a = [1, 2];
var b = [3, 4];
a + b; //"1,23,4"
```

> a + ""(隐式)和前面的 String(a)(显式)之间有一个细微的差别需要注意。根据 ToPrimitive 抽象操作规则，a + ""会对 a 调用 valueOf()方法，然后通过 ToString 抽象操作将返回值转换为字符串，而 String(a)则是直接调用 toString()方法。

### 宽松相等和严格相等

\==允许在相等比较中进行强制类型转换，而===不允许

```
比较运算x==y, 其中x和 y是值，产生true或者false。这样的比较按如下方式进行：
	1. 若Type(x)与Type(y)相同， 则
		a. 若Type(x)为Undefined， 返回true。
		b. 若Type(x)为Null， 返回true。
		c. 若Type(x)为Number， 则
			i. 若x为NaN， 返回false。
			ii. 若y为NaN， 返回false。
			iii. 若x与y为相等数值， 返回true。
			iv. 若x 为 +0 且 y为−0， 返回true。
			v. 若x 为 −0 且 y为+0， 返回true。
			vi. 返回false。
		d. 若Type(x)为String, 则当x和y为完全相同的字符序列（长度相等且相同字符在相同位置）时返回true。 否则， 返回false。
		e. 若Type(x)为Boolean, 当x和y为同为true或者同为false时返回true。 否则， 返回false。
		f. 当x和y为引用同一对象时返回true。否则，返回false。
	2. 若x为null且y为undefined， 返回true。
	3. 若x为undefined且y为null， 返回true。
	4. 若Type(x) 为 Number 且 Type(y)为String， 返回comparison x == ToNumber(y)的结果。
	5. 若Type(x) 为 String 且 Type(y)为Number，返回比较ToNumber(x) == y的结果。
	6. 若Type(x)为Boolean， 返回比较ToNumber(x) == y的结果。
	7. 若Type(y)为Boolean， 返回比较x == ToNumber(y)的结果。
	8. 若Type(x)为String或Number，且Type(y)为Object，返回比较x == ToPrimitive(y)的结果。
	9. 若Type(x)为Object且Type(y)为String或Number， 返回比较ToPrimitive(x) == y的结果。
  10. 返回false。
```

### 几个奇怪的地方

```javascript
[] == ![]; //true
[] == false;
"" == false;

var a = "42";
var b = true;
a == b; //false

//你也许觉得这不可能，因为a不会同时等于2和3。但如果让a.valueOf()每次调用都产生副作用，比如第一次返回2，第二次返回3，就会出现这样的情况。
var i = 2;
Number.prototype.valueOf = function() {
  return i++;
};
var a = new Number(42);
if (a == 2 && a == 3) {
  console.log("Yeah, it happened!");
}
//""、"\n"(或者" "等其他空格组合)等空字符串被ToNumber强制类型转换为0。
0 == "\n"; //true
```

### 抽象关系比较

```
1. 如果 LeftFirst 标志是 true，那么
  a. 让 px 为调用 ToPrimitive(x, hint Number) 的结果。
  b. 让 py 为调用 ToPrimitive(y, hint Number) 的结果。
2. 否则解释执行的顺序需要反转，从而保证从左到右的执行顺序
  a. 让 py 为调用 ToPrimitive(y, hint Number) 的结果。
  b. 让 px 为调用 ToPrimitive(x, hint Number) 的结果。
3. 如果 Type(px) 和 Type(py) 得到的结果不都是 String 类型，那么
  a. 让 nx 为调用 ToNumber(px) 的结果。因为 px 和 py 都已经是基本数据类型（primitive values 也作原始值），其执行顺序并不重要。
  b. 让 ny 为调用 ToNumber(py) 的结果。
  c. 如果 nx 是 NaN，返回 undefined
  d. 如果 ny 是 NaN，返回 undefined
  e. 如果 nx 和 ny 的数字值相同，返回 false
  f. 如果 nx 是 +0 且 ny 是 -0，返回 flase
  g. 如果 nx 是 -0 且 ny 是 +0，返回 false
  h. 如果 nx 是 +∞，返回 fasle
  i. 如果 ny 是 +∞，返回 true
  j. 如果 ny 是 -∞，返回 flase
  k. 如果 nx 是 -∞，返回 true
  l. 如果 nx 数学上的值小于 ny 数学上的值（注意这些数学值都不能是无限的且不能都为 0），返回 ture。否则返回 false。
4. 否则，px 和 py 都是 Strings 类型
  a. 如果 py 是 px 的一个前缀，返回 false。（当字符串 q 的值可以是字符串 p 和一个其他的字符串 r 拼接而成时，字符串 p 就是 q 的前缀。注意：任何字符串都是自己的前缀，因为 r 可能是空字符串。）
  b. 如果 px 是 py 的前缀，返回 true。
  c. 让 k 成为最小的非负整数，能使得在 px 字符串中位置 k 的字符与字符串 py 字符串中位置 k 的字符不相同。（这里必须有一个 k，使得互相都不是对方的前缀）
  d. 让 m 成为字符串 px 中位置 k 的字符的编码单元值。
  e. 让 n 成为字符串 py 中位置 k 的字符的编码单元值。
  f.如果 n<m，返回 true。否则，返回 false。

```

```javascript
var a = { b: 42 };
var b = { b: 43 };

a < b; // false
a == b; // false
a > b; // false

a <= b; // true
a >= b; // true

/*
因为根据规范a <= b被处理为b < a，然后将结果反转。因为b < a的结果为false，所以a <= b的结果为true。
这可能与我们设想的大相径庭，即<=应该是“小于或者等于”，实际上，JavaScript中<=是“不大于”的意思，即a <= b被处理为 !(b < a)。
** 规范设定NaN既不大于也不小于任何其他值。

*/
```
