### async await

- 一个函数如果加上 async ，那么该函数就会返回一个 Promise
- async 就是将函数返回值使用 Promise.resolve() 包裹了下
- await 只能配套 async 使用

```javascript
let a = 0;
let b = async () => {
  console.log(1);
  a = a + (await 10);
  console.log(4);
  console.log("2", a); // -> '2' 10
};
let c = async () => {
  console.log('c1');
  let aa=0;
  aa = aa + (await 10);
  console.log('c2');
  console.log("c", aa); // -> '2' 10
};
b();
c()
console.log(2);
a++;
console.log(3);
console.log("1", a); // -> '1' 1

// 1
// 2
// 3
// 1 1
// 4
// 2 10
```

