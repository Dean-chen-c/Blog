# js 的几种继承方式

### 先提几个问题

- 写一个原型链继承
- 描述 new 一个对象的过程
- zepto 等一些框架如何使用原型链

### 继承

js 的继承实现依靠原型链。
继承的本质就是**复制**，即重写原型对象，代之以一个新类型的**实例**。
**构造函数** **原型** **实例** 三者之间的关系
每个构造函数都有一个原型对象
原型对象都包含一个指向构造函数的指针
实例都包含一个原型对象的指针

```javascript
function a() {}
let aa = new a();

//a的原型
a.prototype;
//构造函数就是a自己
a.prototype.constructor === a;
//实例aa的原型对象a.prototype
a.prototype === aa.__proto__;
```

1. constructor 总是指向类的构造函数
2. **proto**指向父类的原型对象
3. instanceof 可以判断构造函数的继承关系
