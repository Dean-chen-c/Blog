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
  
- Promise顺序处理

  ```js
  function myPromise(value){
  	return ()=>Promise.resolve(value)
  }
  const arr=[myPromise(1),myPromise(2),myPromise(3),myPromise(4)];
  function recordValue(result,value){
      result.push(value);
      return result
  }
  const pushValue=recordValue.bind(null,[]);
  const aa=arr.reduce((a,c)=>{
      return a.then(c).then(pushValue)
  },Promise.resolve())
  aa.then(v=>console.log(v))
  ```

```js
const PENDING = "pending";
const RESOLVED = "resolved";
const REJECTED = "rejected";
function myPromise(fn) {
  const _this = this;
  _this.state = PENDING;
  _this.value = null;
  _this.resolvedCallbacks = [];
  _this.rejectedCallbacks = [];
  function resolve(value) {
    if (_this.state === PENDING) {
      _this.state = RESOLVED;
      _this.value = value;
      _this.resolvedCallbacks.map((cb) => cb(_this.value));
    }
  }
  function reject(value) {
    if (_this.state === PENDING) {
      _this.state = REJECTED;
      _this.value = value;
      _this.rejectedCallbacks.map((cb) => cb(_this.value));
    }
  }
  try {
    fn(resolve, reject);
  } catch (error) {
    reject(error);
  }
}
myPromise.prototype.then = function (onFulfilled, onRejected) {
  const _this = this;
  onFulfilled = typeof onFulfilled === "function" ? onFulfilled : (v) => v;
  onRejected =
    typeof onRejected === "function"
      ? onRejected
      : (r) => {
          throw r;
        };
  if (_this.state === PENDING) {
    // _this.resolvedCallbacks.push(onFulfilled);
    // _this.rejectedCallbacks.push(onRejected);
    return new Promise(function (resolve, reject) {
      _this.resolvedCallbacks.push(function (value) {
        try {
        } catch (error) {}
      });
    });
  }
  if (_this.state === RESOLVED) {
    // onFulfilled(_this.value);
    return new Promise(function (resolve, reject) {
      try {
        const ret = onFulfilled(_this.value);
        if (ret instanceof Promise) {
          ret.then(resolve, reject);
        } else {
          resolve(ret);
        }
      } catch (error) {
        reject(error);
      }
    });
  }
  if (_this.state === REJECTED) {
    // onRejected(_this.value);
    return new Promise(function (resolve, reject) {
      try {
        const ret = onRejected(_this.value);
        if (ret instanceof Promise) {
          ret.then(resolve, reject);
        } else {
          reject(ret);
        }
      } catch (error) {
        reject(error);
      }
    });
  }
};
new MyPromise((resolve, reject) => {
  setTimeout(() => {
    resolve(1);
  }, 0);
}).then((value) => {
  console.log(value);
});

```



```js

```

