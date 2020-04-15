# Set和Map

Set 和 Map 主要的应用场景在于 **数据重组** 和 **数据储存**

#### 集合Set

```js
// 去重数组的重复对象
let arr = [1, 2, 3, 2, 1, 1]
[... new Set(arr)]	// [1, 2, 3]
```

向 Set 加入值的时候，不会发生类型转换，所以`1`和`"1"`是两个不同的值。

```js
let set1 = new Set()
set1.add(5)
set1.add('5')
console.log([...set1])	// [5, "5"]

// 去重数组的重复对象
let arr = [NaN,NaN]
[... new Set(arr)]	// [NaN]
```

Set 实例属性

```js
new Set([NaN,NaN]).size  //元素数量  1
```

Set 实例方法

- add
- delete
- has
- clear
- keys
- values
- entries

Set 很容易实现交集（Intersect）、并集（Union）、差集（Difference）

```
let set1 = new Set([1, 2, 3])
let set2 = new Set([4, 3, 2])

let intersect = new Set([...set1].filter(value=>set2.has(value)))

let union = new Set([...set1,...set2])

let difference = new Set([...set1].filter(value=>!set2.has(value)))
```

#### 字典Map

- 共同点：集合、字典 可以储存不重复的值
- 不同点：集合 是以 [value, value]的形式储存元素，字典 是以 [key, value] 的形式储存

**任何具有 Iterator 接口、且每个成员都是一个双元素的数组的数据结构**都可以当作`Map`构造函数的参数

```js
const set = new Set([
  ['foo', 1],
  ['bar', 2]
]);
const m1 = new Map(set);
m1.get('foo') // 1

const m2 = new Map([['baz', 3],['baz',2]]);
const m3 = new Map(m2);
m3.get('baz') // 2
```

如果读取一个未知的键，则返回`undefined`。

```js
m3.get('bac') // undefined
```

注意，只有对同一个对象的引用，Map 结构才将其视为同一个键。这一点要非常小心。

```js
const map = new Map();
const arr=[2]
map.set([1], 555);
map.set(arr, 333);
map.get([1]) // undefined
map.get(arr) //333
```

Map 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键。这就解决了同名属性碰撞（clash）的问题，我们扩展别人的库的时候，如果使用对象作为键名，就不用担心自己的属性与原作者的属性同名。

如果 Map 的键是一个简单类型的值（数字、字符串、布尔值），则只要两个值严格相等，Map 将其视为一个键，比如`0`和`-0`就是一个键，布尔值`true`和字符串`true`则是两个不同的键。另外，`undefined`和`null`也是两个不同的键。虽然`NaN`不严格相等于自身，但 Map 将其视为同一个键。

```js
let map = new Map();

map.set(-0, 123);
map.get(+0) // 123

map.set(true, 1);
map.set('true', 2);
map.get(true) // 1

map.set(undefined, 3);
map.set(null, 4);
map.get(undefined) // 3

map.set(NaN, 123);
map.get(NaN) // 123
```

#### Map 转 Object

会把非字符串键名转换为字符串键名。

Object 结构提供了 **字符串-值** 对应，Map则提供了 **值-值** 的对应

```js
function mapToObj(map) {
    let obj = Object.create(null)
    for (let [key, value] of map) {
        obj[key] = value
    }
    return obj
}
const map = new Map().set([1], 'An').set({a:1}, 'JS')
mapToObj(map)  // {1: "An", [object object]: "JS"}
```

#### WeakMap

WeakMap 对象是一组键值对的集合，只接受对象作为键名，其中的**键是弱引用对象，而值可以是任意**。

应用：

-  在 DOM 对象上保存相关数据

  ```js
  let wm = new WeakMap(), element = document.querySelector(".element");
  wm.set(element, "data");
  
  let value = wm.get(elemet);
  console.log(value); // data
  
  element.parentNode.removeChild(element);
  element = null;
  ```

  

- 数据缓存

  ```js
  const cache = new WeakMap();
  function countOwnKeys(obj) {
      if (cache.has(obj)) {
          console.log('Cached');
          return cache.get(obj);
      } else {
          console.log('Computed');
          const count = Object.keys(obj).length;
          cache.set(obj, count);
          return count;
      }
  }
  ```

  

- 私有属性

  ```js
  const privateData = new WeakMap();
  
  class Person {
      constructor(name, age) {
          privateData.set(this, { name: name, age: age });
      }
  
      getName() {
          return privateData.get(this).name;
      }
  
      getAge() {
          return privateData.get(this).age;
      }
  }
  
  export default Person;
  ```

  



#### 总结

- Set
  - 成员唯一、无序且不重复
  - [value, value]，键值与键名是一致的（或者说只有键值，没有键名）
  - 可以遍历，方法有：add、delete、has
- WeakSet
  - 成员都是对象
  - 成员都是弱引用，可以被垃圾回收机制回收，可以用来保存DOM节点，不容易造成内存泄漏
  - 不能遍历，方法有add、delete、has
- Map
  - 本质上是键值对的集合，类似集合
  - 可以遍历，方法很多可以跟各种数据格式转换
- WeakMap
  - 只接受对象作为键名（null除外），不接受其他类型的值作为键名
  - 键名是弱引用，键值可以是任意的，键名所指向的对象可以被垃圾回收，此时键名是无效的
  - 不能遍历，方法有get、set、has、delete

