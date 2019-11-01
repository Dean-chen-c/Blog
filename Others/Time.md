```javascript
function sleep(time) {
  return new Promise(resolve => setTimeout(resolve, time));
}
function demo() {
  setInterval(function() {
    console.log(2);
  }, 1000);
  sleep(2000);
}
demo();
```
