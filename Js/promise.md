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
  
  function myPromise(value){
  	return Promise.resolve(value)
  }
  const arr=[myPromise(1),myPromise(2),myPromise(3),myPromise(4)];
  const result = [];
  const list = arr.reduce((a, c) => {
    return a.then(() => c).then((v) => result.push(v));
  }, Promise.resolve());
  list.then(v=>console.log(v))
  ```

```js
const PENDING = "pending";
const RESOLVED = "resolved";
const REJECTED = "rejected";
function myPromise(fn) {
  const _this = this;
  _this.state = PENDING;
  _this.value = null;// 存值
  _this.resolvedCallbacks = [];
  _this.rejectedCallbacks = [];
  function resolve(value) {
    if (_this.state === PENDING) {
      _this.state = RESOLVED;
      _this.value = value;
      _this.resolvedCallbacks.map((cb) => cb());// 执行所有回调
    }
  }
  function reject(value) {
    if (_this.state === PENDING) {
      _this.state = REJECTED;
      _this.value = value;
      _this.rejectedCallbacks.map((cb) => cb());
    }
  }
  try {
    fn(resolve, reject);// 执行fn
  } catch (error) {
    reject(error);
  }
}

myPromise.prototype.then = function (onFulfilled, onRejected) {
  const _this = this;
  // 必须为function
  onFulfilled = typeof onFulfilled === "function" ? onFulfilled : (v) => v;
  onRejected =
    typeof onRejected === "function"
      ? onRejected
      : (r) => {
          throw r;
        };
  // 每次都会返回一个新的promise
  let promise2 = new myPromise((resolve, reject) => {
    //pending 加入回调队列
    if (_this.state === PENDING) {
      _this.resolvedCallbacks.push(() => {
        //异步保证执行顺序
        setTimeout(() => {
          try {
            const x = onFulfilled(_this.value);
            resolutionProcedure(promise2, x, resolve, reject);
          } catch (error) {
            reject(error);
          }
        }, 0);
      });
      _this.rejectedCallbacks.push(() => {
        setTimeout(() => {
          try {
            const x = onRejected(_this.value);
            resolutionProcedure(promise2, x, resolve, reject);
          } catch (error) {
            reject(error);
          }
        }, 0);
      });
    } else if (_this.state === RESOLVED) {
      setTimeout(() => {
        try {
          const x = onFulfilled(_this.value);
          resolutionProcedure(promise2, x, resolve, reject);
        } catch (error) {
          reject(error);
        }
      }, 0);
    } else if (_this.state === REJECTED) {
      setTimeout(() => {
        try {
          const x = onRejected(_this.value);
          resolutionProcedure(promise2, x, resolve, reject);
        } catch (error) {
          reject(error);
        }
      }, 0);
    }
  });
  return promise2;
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

myPromise.defer = myPromise.deferred = function () {
  let dfd = {};
  dfd.promise = new myPromise((resolve, reject) => {
    dfd.resolve = resolve;
    dfd.reject = reject;
  });
  return dfd;
};
myPromise.resolve = function (param) {
  if (param instanceof myPromise) {
    return param;
  }
  return new myPromise((resolve, reject) => {
    if (param && param.then && typeof param.then === "function") {
      setTimeout(() => {
        param.then(resolve, reject);
      });
    } else {
      resolve(param);
    }
  });
};

//Promise.all
Promise.all = function (promises) {
  return new Promise((resolve, reject) => {
    let index = 0;
    let result = [];
    if (promises.length === 0) {
      resolve(result);
    } else {
      for (let i = 0; i < promises.length; i++) {
        Promise.resolve(promises[i]).then(
          (data) => {
            result[i] = data;
            index++;
            if (index === promises.length) {
              resolve(result);
            }
          },
          (err) => {
            reject(err);
            return;
          }
        );
      }
    }
  });
};

//Promise.reject
Promise.reject = function (reason) {
  return new Promise((resolve, reject) => {
    reject(reason);
  });
};

//Promise.prototype.catch
Promise.prototype.catch = function (onRejected) {
  return this.then(null, onRejected);
};

module.exports = myPromise;

/*
	npm install -g promises-aplus-tests
	promises-aplus-tests index.js
*/
// var dummy = { dummy: "dummy" };

// var resolved = function (value) {
//   var d = myPromise.deferred();
//   d.resolve(value);
//   return d.promise;
// };
// var promise = resolved(dummy).then(function () {
//   return promise;
// });
// promise.then(null, function (reason) {
//   console.log("typeError");
// });

```


