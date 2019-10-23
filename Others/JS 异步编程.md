# JS 异步编程

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

```javascript
```

```javascript
```

```javascript
```

```javascript
```

```javascript
```

```javascript
```
