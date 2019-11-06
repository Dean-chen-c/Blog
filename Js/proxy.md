# Proxy

> Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。

---

通过 Proxy 来实现一个数据响应式

```javascript
let onWatch = (obj, setBind, getLogger) => {
  let handler = {
    get(target, property, receiver) {
      getLogger(target, property);
      return Reflect.get(target, property, receiver);
    },
    set(target, property, value, receiver) {
      setBind(value, property);
      return Reflect.set(target, property, value, receiver);
    }
  };
  return new Proxy(obj, handler);
};
let obj = { a: 1 };
let p = onWatch(
  obj,
  (v, property) => {
    console.log(`监听到属性${property}改变为${v}`);
  },
  (target, property) => {
    console.log(`'${property}' = ${target[property]}`);
  }
);
p.a = 2; // 监听到属性a改变
p.a; // 'a' = 2
```

### Reflect 使用

#### Reflect.get(target, name, receiver)

```javascript
var myObject = {
  foo: 1,
  bar: 2,
  get baz() {
    return this.foo + this.bar;
  }
};

Reflect.get(myObject, "foo"); // 1
Reflect.get(myObject, "bar"); // 2
Reflect.get(myObject, "baz"); // 3

var myObject = {
  foo: 1,
  bar: 2,
  get baz() {
    return this.foo + this.bar;
  }
};

var myReceiverObject = {
  foo: 4,
  bar: 4
};

Reflect.get(myObject, "baz", myReceiverObject); // 8
```

#### Reflect.set(target, name, value, receiver)

```javascript
var myObject = {
  foo: 1,
  set bar(value) {
    return (this.foo = value);
  }
};

myObject.foo; // 1

Reflect.set(myObject, "foo", 2);
myObject.foo; // 2

Reflect.set(myObject, "bar", 3);
myObject.foo; // 3

var myObject = {
  foo: 4,
  set bar(value) {
    return (this.foo = value);
  }
};

var myReceiverObject = {
  foo: 0
};

Reflect.set(myObject, "bar", 1, myReceiverObject);
myObject.foo; // 4
myReceiverObject.foo; // 1
```
