#### 跨Tab页通信方式

- 使用localStorage

  ```js
  function message_broadcast(message)
  {
      localStorage.setItem('message',JSON.stringify(message));
      localStorage.removeItem('message');
  }
  window.addEventListener('storage', function (e) {
      if (e.key === 'message') {
          if (!e.newValue) return;
          let message = JSON.parse(e.newValue);
          console.log(message)
      }
  });
  ```

- 

