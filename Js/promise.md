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
    // if (value instanceof myPromise) {
    //   return value.then(resolve, reject);
    // }
    setTimeout(() => {
      if (_this.state === PENDING) {
        _this.state = RESOLVED;
        _this.value = value;
        _this.resolvedCallbacks.map((cb) => cb());
      }
    }, 0);
  }
  function reject(value) {
    setTimeout(() => {
      if (_this.state === PENDING) {
        _this.state = REJECTED;
        _this.value = value;
        _this.rejectedCallbacks.map((cb) => cb());
      }
    }, 0);
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
    return (promise2 = new Promise((resolve, reject) => {
      _this.resolvedCallbacks.push(() => {
        // setTimeout(() => {
        try {
          const x = onFulfilled(_this.value);
          resolutionProcedure(promise2, x, resolve, reject);
        } catch (error) {
          reject(error);
        }
        // }, 0);
      });
      _this.rejectedCallbacks.push(() => {
        // setTimeout(() => {
        try {
          const x = onRejected(_this.value);
          resolutionProcedure(promise2, x, resolve, reject);
        } catch (error) {
          reject(error);
        }
        // }, 0);
      });
    }));
  }
  if (_this.state === RESOLVED) {
    return (promise2 = new Promise((resolve, reject) => {
      setTimeout(() => {
        try {
          const x = onFulfilled(_this.value);
          resolutionProcedure(promise2, x, resolve, reject);
        } catch (error) {
          reject(error);
        }
      }, 0);
    }));
  }
  if (_this.state === REJECTED) {
    return (promise2 = new Promise((resolve, reject) => {
      setTimeout(() => {
        try {
          const x = onRejected(_this.value);
          resolutionProcedure(promise2, x, resolve, reject);
        } catch (error) {
          reject(error);
        }
      }, 0);
    }));
  }
};

function resolutionProcedure(promise2, x, resolve, reject) {
  if (promise2 === x) {
    return reject(new TypeError("Chaining cycle"));
  }
  if (x && (typeof x === "function" || typeof x === "object")) {
    let called = false;
    try {
      const then = x.then;
      if (typeof then === "function") {
        then.call(
          x,
          (y) => {
            if (called) return;
            called = true;
            resolutionProcedure(promise2, y, resolve, reject);
          },
          (r) => {
            if (called) return;
            called = true;
            reject(r);
          }
        );
      } else {
        if (called) return;
        called = true;
        resolve(x);
      }
    } catch (error) {
      if (called) return;
      called = true;
      reject(error);
    }
  } else {
    resolve(x);
  }
}

// myPromise.defer = myPromise.deferred = function () {
//   let dfd = {};
//   dfd.promise = new myPromise((resolve, reject) => {
//     dfd.resolve = resolve;
//     dfd.reject = reject;
//   });
//   return dfd;
// };

// module.exports = myPromise;

/*
	npm install -g promises-aplus-tests
	promises-aplus-tests index.js
*/

```


