```js
console.log('1');

setTimeout(function() {
console.log('2');
process.nextTick(function() {
console.log('3');
})
new Promise(function(resolve) {
console.log('4');
resolve();
}).then(function() {
console.log('5')
})
})
process.nextTick(function() {
console.log('6');
})
new Promise(function(resolve) {
console.log('7');
resolve();
}).then(function() {
console.log('8')
})

setTimeout(function() {
console.log('9');
process.nextTick(function() {
console.log('10');
})
new Promise(function(resolve) {
console.log('11');
resolve();
}).then(function() {
console.log('12')
})
```



    1 7 8 2 4 9 11 5 12



### Promise

- Promise.resolve()
  - like newPromise(resolve=>resolve())
- Promise.reject()
- Promise.all()
  - 并行 一个reject或者全部resolve结束
- Promise.race
  - 第一个成功后返回结果，其它promise也没取消