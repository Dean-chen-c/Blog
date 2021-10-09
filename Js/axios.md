# Axios

#### CancelToken

```js
const CancelToken = CancelToken;
const source = CancelToken.source();
// axios.post('/user/12345', {
//   name: 'new name'
// }, {
//   cancelToken: source.token
// })
const cancelToken=source.token;

source.cancel('Operation canceled by the user.');


function CancelToken(executor) {
  if (typeof executor !== 'function') {
    throw new TypeError('executor must be a function.');
  }

  var resolvePromise;
  this.promise = new Promise(function promiseExecutor(resolve) {
    resolvePromise = resolve;
  });

  var _this = this;
  executor(function cancel(message) {
    if (_this.reason) {
      return;
    }

    _this.reason = new Cancel(message);
    resolvePromise(_this.reason);
  });
}

CancelToken.prototype.throwIfRequested = function throwIfRequested() {
  if (this.reason) {
    throw this.reason;
  }
};

CancelToken.source = function source() {
  var cancel;
  var token = new CancelToken(function executor(c) {
    cancel = c;
  });
  return {
    token: token,
    cancel: cancel
  };
};

if (config.cancelToken) {
  // Handle cancellation
  config.cancelToken.promise.then(function onCanceled(cancel) {
    if (req.aborted) return;

    req.abort();
    reject(cancel);
  });
}

if (config.timeout) {
  req.setTimeout(config.timeout, function handleRequestTimeout() {
    req.abort();
    reject(createError('timeout of ' + config.timeout + 'ms exceeded', config, 'ECONNABORTED', req));
  });
}
```

