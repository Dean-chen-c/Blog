# This

this 就是一个指针，指向调用函数的对象。

this 的绑定规则:

- 默认绑定
- 隐式绑定
- 硬绑定
- new 绑定

### 默认绑定

```javascript
function sayHi() {
  console.log("Hello,", this.name);
}
var name = "World";
sayHi();
//Hello,World
```

在调用 Hi()时，应用了默认绑定，this 指向全局对象（非严格模式下），严格模式下，this 指向 undefined，undefined 上没有 this 对象，会抛出错误。

### 隐式绑定

sayHi 函数声明在外部，严格来说并不属于 person，但是在调用 sayHi 时,调用位置会使用 person 的上下文来引用函数，

```javascript
function sayHi() {
  console.log("Hello,", this.name);
}
var person = {
  name: "World",
  sayHi: sayHi
};
var name = "H";
person.sayHi();
//Hello, World
```

Hi 直接指向了 sayHi 的引用，在调用的时候，跟 person 没有半毛钱的关系

```javascript
function sayHi() {
  console.log("Hello,", this.name);
}
var person = {
  name: "World",
  sayHi: sayHi
};
var name = "H";
var Hi = person.sayHi;
Hi();
//Hello, H
```

隐式绑定的丢失是发生在**回调函数**中(事件回调也是其中一种)

```javascript
function sayHi() {
  console.log("Hello,", this.name);
}
var person1 = {
  name: "World",
  sayHi: function() {
    setTimeout(function() {
      console.log("Hello,", this.name);
    });
  }
};
var person2 = {
  name: "World2",
  sayHi: sayHi
};
var name = "H";
person1.sayHi();
//Hello, H
setTimeout(person2.sayHi, 100);
//Hello, H
setTimeout(function() {
  person2.sayHi();
}, 200);

//Hello, World2
```

### 显式绑定

就是通过 call,apply,bind 的方式，显式的指定 this 所指向的对象。call 和 apply 的作用一样，只是传参方式不同。call 参数列表 apply 多个参数的数组

```
foo.apply(bar, [1, 2, 3]); // 数组将会被扩展，如下所示
foo.call(bar, 1, 2, 3); // 传递到foo的参数是：a = 1, b = 2, c = 3
```

```js
Function.prototype.mycall = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('a.mycall is not function');
  }
  const c = context || window;
  const arg = [...arguments].slice(1);
  c[Symbol.for('call')] = this;
  c[Symbol.for('call')](...arg);
    
  delete c[Symbol.for('call')];
};
Function.prototype.myapply = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('a.myapply is not function');
  }
  const c = context || window;
  const arg = arguments[1] || [];
  c[Symbol.for('apply')] = this;
  c[Symbol.for('apply')](...arg);

  delete c[Symbol.for('apply')];
};
Function.prototype.mybind = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('...');
  }
  const _ = this;
  const arg = [...arguments].slice(1);
  return function F() {
    if (this instanceof F) {
      return new _(...arg.concat(...arguments));
    } else {
      return _.myapply(context, arg.concat(...arguments));
    }
  };
};
```

```javascript
function sayHi() {
  console.log("Hello,", this.name);
}
var person = {
  name: "World",
  sayHi: sayHi
};
var name = "H";
var Hi = person.sayHi;
Hi.call(person); //Hi.apply(person)
//Hello, World
var Hi = function(fn) {
  fn();
};
Hi.call(person, person.sayHi);
//Hello, H
var Hi = function(fn) {
  fn.call(this);
};
Hi.call(person, person.sayHi);
//Hello, World
```

### new 绑定

> 使用 new 来调用函数，会自动执行下面的操作：

1. 创建一个空对象，构造函数中的 this 指向这个空对象
2. 这个新对象被执行原型连接
3. 执行构造函数方法，属性和方法被添加到 this 引用的对象中
4. 如果构造函数中没有返回其它对象，那么返回 this，即创建的这个的新对象，否则，返回构造函数中返回的对象。

```javascript
function _new() {
  let target = {}; //创建的新对象
  //第一个参数是构造函数
  let [constructor, ...args] = [...arguments];
  //执行[[原型]]连接;target 是 constructor 的实例
  target.__proto__ = constructor.prototype;
  //执行构造函数，将属性或方法添加到创建的空对象上
  let result = constructor.apply(target, args);
  if (result && (typeof result == "object" || typeof result == "function")) {
    //如果构造函数执行的结构返回的是一个对象，那么返回这个对象
    return result;
  }
  //如果构造函数返回的不是一个对象，返回创建的新对象
  return target;
}
function _new2() {
  let Constructor = [].shift.call(arguments);
  const obj = new Object();
  obj.__proto__ = Constructor.prototype;
  let result = Constructor.call(obj, ...arguments);
  if (result && (typeof result == "object" || typeof result == "function")) {
    return result;
  }
  return obj;
}
function _new3(Obj, ...args) {
  var obj = Object.create(Obj.prototype); //使用指定的原型对象及其属性去创建一个新的对象
  let result = Obj.apply(obj, args); // 绑定 this 到obj, 设置 obj 的属性
  if (result && (typeof result == "object" || typeof result == "function")) {
    return result;
  }
  return obj; // 返回实例
}
```

### 绑定优先级

new 绑定 > 显式绑定 > 隐式绑定 > 默认绑定

我们将 null 或者是 undefined 作为 this 的绑定对象传入 call、apply 或者是 bind,这些值在调用时会被忽略，实际应用的是默认绑定规则。

### 箭头函数

箭头函数没有自己的 this，它的 this 继承于外层代码库中的 this。

1. 函数体内的 this 对象，继承的是外层代码块的 this。
2. 不可以当作构造函数，也就是说，不可以使用 new 命令，否则会抛出一个错误。
3. 不可以使用 arguments 对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。
4. 不可以使用 yield 命令，因此箭头函数不能用作 Generator 函数。
5. 箭头函数没有自己的 this，所以不能用 call()、apply()、bind()这些方法去改变 this 的指向.

```javascript
var obj = {
  a: () => {
    console.log(this);
  },
  b: function() {
    console.log(this);
    return () => {
      console.log(this);
    };
  },
  c: function() {
    return function() {
      console.log(this);
      return () => {
        console.log(this);
      };
    };
  }
};
let b = obj.b(); //输出obj对象
b(); //输出obj对象
let c = obj.c();
let fun1 = c(); //输出window
fun1(); //输出window
obj.a(); //输出window
```

### 题目

```javascript
var number = 5;
var obj = {
  number: 3,
  fn: (function() {
    var number;
    this.number *= 2;
    number = number * 2;
    number = 3;
    return function() {
      var num = this.number;
      this.number *= 2;
      console.log(num);
      number *= 3;
      console.log(number);
    };
  })()
};
var myFun = obj.fn;
myFun.call(null);
obj.fn();
console.log(window.number);
```

```
10
9
3
27
20

```
