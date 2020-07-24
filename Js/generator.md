# generator

特点：

- function\*(){} 和 yield 和 next()
- Generator 函数是可以控制 iterator（迭代器）的函数。并在任何时候都可以暂停和恢复。
- 调用 Generator 函数后，该函数并**不执行**，返回的也不是函数运行结果，而是一个指向内部状态的指针对象
- yield 表达式是暂停执行的标记，而 next 方法可以恢复执行。
- next 函数的返回值与 yield 之后的值有关，而 yield 表达式的值却与此无关。
- next 函数的参数即为上一个 yield 表达式的返回值,没有参数就是 undefined。

### for of

```javascript
function* fibonacci() {
  let [pre, cur] = [0, 1];
  while (true) {
    [pre, cur] = [cur, pre + cur];
    yield cur;
  }
}
for (let n of fibonacci()) {
  if (n > 1000) break;
  console.log(n);
}
```

```javascript
function run(genarator) {
  console.log(0);
  const iterator = genarator();
  console.log(1);
  const iteration = iterator.next();
  console.log(iteration);
  console.log(4);
  const promise = iteration.value;
  console.log(5);
  promise.then((res) => {
    console.log(6);
    const anotherIteration = iterator.next(res);
    console.log(anotherIteration);
    console.log(7);
    const p = anotherIteration.value;
    console.log(8);
    p.then((r) => {
      console.log(9);
      const aaI = iterator.next(r);
      console.log(aaI);
      return aaI;
    });
  });
}
function run(genarator) {
  const iterator = genarator();
  function iterate(iteration) {
    if (iteration.done) return iteration.value;
    return promise.then((x) => iterate(iterator.next(x)));
  }
  return iterate(iterator.next());
}
run(function* () {
  console.log(2);
  const uri = "http://47.112.101.19:9990/sw-cms/api/queryAllChannel/cn";
  console.log(3);
  const res = yield fetch(uri);
  console.log(res);
  const post = yield res.json();
  console.log(post);
});
```

```javascript
function* gen() {
  console.log(2);
  const uri = "http://47.112.101.19:9990/sw-cms/api/queryAllChannel/cn";
  console.log(3);
  const res = yield fetch(uri);
  console.log(res);
  const post = yield res.json();
  console.log(post);
}
const g = gen();
const c = g.next();
c.value.then((res) => {
  const c = g.next(res);
  c.value.then((res) => {
    g.next(res);
  });
});
```

```javascript
let count = 0;
function* gen() {
  console.log("--gen--");
  const res = yield asycf();
  console.log("res:" + res);
  const res1 = yield asycf(res);
  console.log("res1:" + res1);
  const res2 = yield asycf(res1);
  console.log("res2:" + res2);
}

function asycf() {
  console.log("--asycf--");
  setTimeout(() => {
    console.log("--setTimeout--");
    g.next(count++);
  });
}
const g = gen();
g.next();

/*

--gen--
--asycf--
--setTimeout--
res:0
--asycf--
--setTimeout--
res1:1
--asycf--
--setTimeout--
res2:2

*/
```

```js
function fetchData(url) {
  return function (cb) {
    if (url !== "3") {
      console.log(url);
      setTimeout(function () {
        cb(null, { status: 200, data: url });
      }, 1000);
    } else {
      console.log(2);
      cb(new Error("ww"));
      // throw new Error("www");
    }
  };
}
function* gen() {
  const r1 = yield fetch("https://v1.jinrishici.com/all.json");
  const json1 = yield r1.json();
  const r2 = yield fetch("https://v1.jinrishici.com/rensheng.json");
  const json2 = yield r2.json();
  const r3 = yield fetch("https://v1.jinrishici.com/shuqing/libie.json");
  const json3 = yield r3.json();
  var r4 = yield fetchData("1");
  var r5 = yield fetchData("2");
  var r6 = yield fetchData("3");
  return [json1, json2, json3, r1, r2, r3];
}
```

```js
function isPromise(obj) {
  return "function" == typeof obj.then;
}
function toPromise(obj) {
  if (isPromise(obj)) return obj;
  if ("function" == typeof obj) return thunkToPromise(obj);
  return obj;
}
function thunkToPromise(fn) {
  return new Promise(function (resolve, reject) {
    fn(function (err, res) {
      if (err) return reject(err);
      resolve(res);
    });
  });
}
```

```js
//只支持yield Promise
function run1(gen) {
  const g = gen();
  function next(data) {
    const r = g.next(data);
    if (r.done) return;
    r.value.then((d) => next(d));
  }
  next();
}
run1(gen);
//只支持yield function
var g = gen();
const r = g.next();
r.value(function (d) {
  const r2 = g.next(d);
  r2.value(function (d) {
    g.next(d);
  });
});
//两者都支持
function run(gen) {
  const g = gen();
  function next(data) {
    const r = g.next(data);
    if (r.done) return;
    if (isPromise(r.value)) {
      r.value.then((d) => next(d));
    } else {
      r.value(next);
    }
  }
  next();
}
//支持返回值，错误处理
function run(gen) {
  const g = gen();
  const promise = new Promise((resolve, reject) => {
    function next(data) {
      let r;
      try {
        r = g.next(data);
      } catch (error) {
        reject(error);
      }
      if (r.done) return resolve(r.value);

      const v = toPromise(r.value);
      v.then((d) => next(d)).catch((e) => reject(e));
    }
    next();
  });
  return promise;
}
//边界判断 优化
function run(gen) {
  return new Promise((resolve, reject) => {
    if (typeof gen === "function") gen = gen();
    if (!gen || typeof gen.next !== "function") return resolve(gen);
    onFulfilled();
    function onFulfilled(res) {
      let r;
      try {
        r = gen.next(res);
      } catch (error) {
        return reject(error);
      }
      next(r);
    }
    function onRejected(err) {
      let r;
      try {
        ret = gen.throw(err);
      } catch (error) {
        return reject(error);
      }
      next(r);
    }
    function next(r) {
      if (r.done) return resolve(r.value);
      var value = toPromise(r.value);
      if (value && isPromise(value)) return value.then(onFulfilled, onRejected);
      return onRejected(
        new TypeError(
          "You may only yield a function, promise " +
            'but the following object was passed: "' +
            String(ret.value) +
            '"'
        )
      );
    }
  });
}
```

```javascript
(function () {
  let ContinueSentinel = {};

  let mark = function (genFun) {
    let generator = Object.create({
      next: function (arg) {
        return this._invoke("next", arg);
      },
    });
    genFun.prototype = generator;
    return genFun;
  };

  function wrap(innerFn, outerFn, self) {
    let generator = Object.create(outerFn.prototype);

    let context = {
      done: false,
      method: "next",
      next: 0,
      prev: 0,
      abrupt: function (type, arg) {
        let record = {};
        record.type = type;
        record.arg = arg;

        return this.complete(record);
      },
      complete: function (record, afterLoc) {
        if (record.type === "return") {
          this.rval = this.arg = record.arg;
          this.method = "return";
          this.next = "end";
        }

        return ContinueSentinel;
      },
      stop: function () {
        this.done = true;
        return this.rval;
      },
    };

    generator._invoke = makeInvokeMethod(innerFn, context);

    return generator;
  }

  function makeInvokeMethod(innerFn, context) {
    let state = "start";

    return function invoke(method, arg) {
      if (state === "completed") {
        return { value: undefined, done: true };
      }

      context.method = method;
      context.arg = arg;

      while (true) {
        state = "executing";

        let record = {
          type: "normal",
          arg: innerFn.call(self, context),
        };

        if (record.type === "normal") {
          state = context.done ? "completed" : "yield";

          if (record.arg === ContinueSentinel) {
            continue;
          }

          return {
            value: record.arg,
            done: context.done,
          };
        }
      }
    };
  }

  window.regeneratorRuntime = {};

  window.regeneratorRuntime.wrap = wrap;
  window.regeneratorRuntime.mark = mark;
})();

// let _marked = window.regeneratorRuntime.mark(helloWorldGenerator);

// function helloWorldGenerator() {
//   return window.regeneratorRuntime.wrap(
//     function helloWorldGenerator$(_context) {
//       while (1) {
//         switch ((_context.prev = _context.next)) {
//           case 0:
//             _context.next = 2;
//             return 'hello';

//           case 2:
//             _context.next = 4;
//             return 'world';

//           case 4:
//             return _context.abrupt('return', 'ending');

//           case 5:
//           case 'end':
//             return _context.stop();
//         }
//       }
//     },
//     _marked,
//     this
//   );
// }

// let hw = helloWorldGenerator();

// console.log(hw.next());
// console.log(hw.next());
// console.log(hw.next());
// console.log(hw.next());

// function* generateRandoms(max) {
//   max = max || 1;

//   while (true) {
//     let newMax = yield Math.random() * max;
//     let newMin = yield Math.random() * 0.1;
//     if (newMax !== undefined || newMin !== undefined) {
//       max = newMax;
//       console.log(newMin);
//     }
//   }
// }
// let a = generateRandoms();
let _marked = window.regeneratorRuntime.mark(helloWorldGenerator);

function helloWorldGenerator(max) {
  let newMax;
  return window.regeneratorRuntime.wrap(function helloWorldGenerator$(
    _context
  ) {
    while (1) {
      switch ((_context.prev = _context.next)) {
        case 0:
          max = max || 1;
        // break;
        // eslint-disable-next-line no-fallthrough
        case 1:
          _context.next = 4;
          return Math.random() * max;

        case 4:
          newMax = _context.arg;

          if (newMax !== undefined) {
            max = newMax;
          }

          _context.next = 1;
          break;

        case 8:
        case "end":
          return _context.stop();
        default:
          return _context.stop();
      }
    }
  },
  _marked);
}
let hw = helloWorldGenerator();
```
