### let

```js
for (let i = 0; i < 3; i++) {
  //console.log(i); //会报错哦 ，暂时性死区
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// abc

for (var i = 0; i < 3; i++) {
  console.log(i);
  var i = 'abc';
  console.log(i);
}
// 0
// abc
```

