# JS 异步编程

js为什么是单线程的？

- 如果是多线程，一个线程想修改一个dom元素，一个想删除这个dom元素。就会出现问题。
- 现在也允许多个js线程，比如worker,但是子线程是不能操作dom的

单线程就带来问题，前一个没有执行完，后面的代码就会一直等待，这样是不可取的。

异步的实现方式就是事件循环Event Loop

### 回调函数（Callback）

一个函数作为参数需要依赖另一个函数执行调用

回调函数有一个致命的弱点，就是容易写出回调地狱（Callback hell）

- 嵌套函数存在耦合性，一旦有所改动，就会牵一发而动全身
- 嵌套函数一多，就很难处理错误

```javascript
ajax(url, () => {
  // 处理逻辑
  ajax(url1, () => {
    // 处理逻辑
    ajax(url2, () => {
      // 处理逻辑
    });
  });
});
```





#### 错误捕获

try catch 无法捕获异步任务中的错误

- Event Loop有关，try catch同步代码执行，遇到异步代码，挂起，当异步任务完成加入到Task Queue,之前的执行栈已经为空，上下文已经发生改变，无法捕获 task 的错误。

#### promise 的异常捕获

都不能捕获到 error，因为 promise 内部的错误不会冒泡出来，而是被 promise 吃掉了，只有通过 promise.catch 才可以捕获，所以用 Promise 一定要写 catch 啊。

```js
function main1() {
  try {
    new Promise(() => {
      throw new Error('promise1 error')
    })
  } catch(e) {
    console.log(e.message);
  }
}

function main2() {
  try {
    Promise.reject('promise2 error');
  } catch(e) {
    console.log(e.message);
  }
}
```

#### async/await 的异常捕获

```js
const fetchFailure = () => new Promise((resolve, reject) => {
  setTimeout(() => {// 模拟请求
    if(1) reject('fetch failure...');
  })
})

async function main () {
  try {
    const res = await fetchFailure();
    console.log(res, 'res');
  } catch(e) {
    console.log(e, 'e.message');
  }
}
main();

```

```js
const fetchFailure = () => new Promise((resolve, reject) => {
  setTimeout(() => {// 模拟请求
    if(1) reject('fetch failure...');
  })
})
const handle = (fn) => {
  return Promise.resolve(fn()).then(data=>[null,data]).catch(err=>[err,null])
}

async function main () {
    const [err,res] = await handle(fetchFailure);
    if(err){
        console.log(err);
        return;
    }
    console.log(res, 'res');
}
```

