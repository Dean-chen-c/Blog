# js 的几种继承方式

### 先提几个问题

- 写一个原型链继承
- 描述 new 一个对象的过程(见 this 指向)
- zepto 等一些框架如何使用原型链

### 继承

js 的继承实现依靠原型链。
继承的本质就是**复制**，即重写原型对象，代之以一个新类型的**实例**。
**构造函数** **原型** **实例** 三者之间的关系
每个构造函数都有一个原型对象
原型对象都包含一个指向构造函数的指针
实例都包含一个原型对象的指针

- 原型链继承
  - 不能传父类参数
  - 不能多继承
  - 实例既是子类的实例，也是父类的实例

```javascript
function p(name) {
  this.name = name;
  this.book = ["c", "c++", "c#"];
}
p.prototype.getName = function() {
  return this.name;
};
function c() {}
c.prototype = new p("p");
c.prototype.constructor = c;
const c1 = new c("cc");
const c2 = new c("ccc");
c1.getName(); //p

c2.getName(); //p
c1.book.push("js");
c2.book; // ["c", "c++", "c#", "js"]
c1 instanceof p; //true
c1 instanceof c; //true
```

- call 继承
  - 实例只是子类的实例
  - 可以实现多继承，可以传参
  - 不能继承原型上的属性
  - 每个子类都有父类函数的副本

```javascript
function p(name) {
  this.name = name;
  this.book = ["c", "c++", "c#"];
}
p.prototype.getName = function() {
  return this.name;
};
function c(name, age) {
  p.call(this, name);
  this.age = age;
}
const c1 = new c("cc", 16);
const c2 = new c("ccc", 18);
c1.getName(); //TypeError
c1.book.push("js");
c2.book; // ["c", "c++", "c#"]
c1 instanceof p; //false
c1 instanceof c; //true
```

- 组合继承
  - 可以继承实例属性/方法，也可以继承原型属性/方法
  - 既是子类的实例，也是父类的实例
  - 可传参 函数可复用
  - 调用了两次父类构造函数

```javascript
function p(name) {
  this.name = name;
  this.book = ["c", "c++", "c#"];
}
p.prototype.getName = function() {
  return this.name;
};
function c(name, age) {
  p.call(this, name);
  this.age = age;
}
//实际上把父类私有也带过来了
//但是子类实例访问的时候首先访问子类的私有属性
c.prototype = new p("p");
c.prototype.constructor = c;
const c1 = new c("cc", 16);
const c2 = new c("ccc", 18);
c1.getName(); //cc
c2.getName(); //ccc
c1.book.push("js");
c2.book; // ["c", "c++", "c#"]
c1 instanceof p; //true
c1 instanceof c; //true
c1.name; //"cc"
c1.book; //["c", "c++", "c#","js"]
c1.__proto__.name; //"p"
c1.__proto__.book; //["c", "c++", "c#"]
```

- 组合继承(优化)
  - 改变了父类 constructor

```javascript
function p(name) {
  this.name = name;
  this.book = ["c", "c++", "c#"];
}
p.prototype.getName = function() {
  return this.name;
};
function c(name, age) {
  p.call(this, name);
  this.age = age;
}

c.prototype = p.prototype;
c.prototype.constructor = c;
const c1 = new c("cc", 16);
const c2 = new c("ccc", 18);
c1.getName(); //cc
c2.getName(); //ccc
c1.book.push("js");
c2.book; // ["c", "c++", "c#"]
c1 instanceof p; //true
c1 instanceof c; //true
c1.name; //"cc"
c1.book; //["c", "c++", "c#","js"]
c1.__proto__.name; //undefined
c1.__proto__.book; //undefined
```

- 组合继承(优化)

```javascript
function p(name) {
  this.name = name;
  this.book = ["c", "c++", "c#"];
}
p.prototype.getName = function() {
  return this.name;
};
function c(name, age) {
  p.call(this, name);
  this.age = age;
}

c.prototype = Object.create(p.prototype, { constructor: { value: c } });
const c1 = new c("cc", 16);
const c2 = new c("ccc", 18);
c1.getName(); //cc
c2.getName(); //ccc
c1.book.push("js");
c2.book; // ["c", "c++", "c#"]
c1 instanceof p; //true
c1 instanceof c; //true
c1.name; //"cc"
c1.book; //["c", "c++", "c#","js"]
c1.__proto__.name; //undefined
c1.__proto__.book; //undefined
```

- class 继承

```javascript
class P {
  constructor(name) {
    this.name = name;
    this.book = ["c", "c++", "c#"];
  }
  getName() {
    return this.name;
  }
}

class C extends P {
  constructor({ name, age }) {
    super(name);
    this.age = age;
  }
}

const c1 = new C({ name: "cc", age: 16 });
const c2 = new C({ name: "ccc", age: 18 });
c1.getName(); //cc
c2.getName(); //ccc
c1.book.push("js");
c2.book; // ["c", "c++", "c#"]
c1 instanceof p; //true
c1 instanceof c; //true
c1.name; //"cc"
c1.book; //["c", "c++", "c#","js"]
c1.__proto__.name; //undefined
c1.__proto__.book; //undefined
```

- babel

```javascript
function _inherits(subClass, superClass) {
  if (typeof superClass !== "function" && superClass !== null) {
    throw new TypeError("Super expression must either be null or a function");
  }
  subClass.prototype = Object.create(superClass && superClass.prototype, {
    constructor: { value: subClass, writable: true, configurable: true }
  });
  if (superClass) _setPrototypeOf(subClass, superClass);
}
function _setPrototypeOf(o, p) {
  _setPrototypeOf =
    Object.setPrototypeOf ||
    function _setPrototypeOf(o, p) {
      o.__proto__ = p;
      return o;
    };
  return _setPrototypeOf(o, p);
}
```
