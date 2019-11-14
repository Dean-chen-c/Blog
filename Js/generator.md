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
  promise.then(res => {
    console.log(6);
    const anotherIteration = iterator.next(res);
    console.log(anotherIteration);
    console.log(7);
    const p = anotherIteration.value;
    console.log(8);
    p.then(r => {
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
    return promise.then(x => iterate(iterator.next(x)));
  }
  return iterate(iterator.next());
}
run(function*() {
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
c.value.then(res => {
  const c = g.next(res);
  c.value.then(res => {
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
