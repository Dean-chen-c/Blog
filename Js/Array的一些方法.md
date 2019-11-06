map 作用是生成一个新数组，遍历原数组，将每个元素拿出来做一些变换然后放入到新的数组中。

```javascript
[1, 2, 3].map(v => v + 1); // -> [2, 3, 4]
```

filter 的作用也是生成一个新数组，在遍历数组的时候将返回值为 true 的元素放入新数组，我们可以利用这个函数删除一些不需要的元素

```javascript
let array = [0, 1, 2, 4, 6];
let newArray = array.filter(item => item !== 6);
console.log(newArray); // [1, 2, 4]
let newArray1 = array.filter(Boolean);
console.log(newArray1); // [1, 2, 4, 6]
```

reduce

- 可以将数组中的元素通过回调函数最终转换为一个值。

```javascript
const arr = [1, 2, 3];
const sum = arr.reduce((acc, current) => acc + current, 0);
console.log(sum);
```

- 实现 map 的作用

```javascript
const arr = [1, 2, 3];
const double = arr.reduce((acc, current) => acc.concat(current * 2), []);
console.log(double);
```

- 数组转对象

```javascript
const arr = [1, 2, 3];
const toObject = arr.reduce((acc, current) => {
  acc[current] = current;
  return acc;
}, {});
console.log(toObject);
```

- 统计字符串字符个数

```javascript
"ssadsa".split("").reduce((a, b) => {
  if (typeof a[b] === "undefined") {
    a[b] = 1;
  } else {
    a[b]++;
  }
  return a;
}, {});
```

push pop unshift shift

```javascript
```

fill

- 生成一个长度 10 的数组，并且全部填充 0

```javascript
const arr = Array(10).fill(0);
```

every

- 妙用如果存在 false 直接跳出循环

```javascript
[1, 3, 4, 5, -1, 4, 4].every(item => {
  console.log(item);
  return item > 0;
});
```
