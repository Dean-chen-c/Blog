#### Event Loop

首先要明白几个知识点

- JS单线程
- 任务队列Task Queue（async），也可以叫callback queue
- 执行栈Task（sync）
- 浏览器进程
  - 渲染进程
    - GUI线程
    - JS线程
    - 定时器线程
    - 事件触发线程
    - 异步HTTP线程





#### NODE

- process.nextTick 
  - 当前"执行栈"的尾部
  - 下一次Event Loop（主线程读取"任务队列"）之前
  - 触发callback
- setImmediate
  - 是在当前"任务队列"的尾部添加事件