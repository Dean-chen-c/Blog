```js
function get(obj, str) {
  return str.split('.').reduce((x, y) => {
    return x !== undefined && x !== null ? x[y] : x
  },
  obj);
}
const obj = {
  a: {
    b: ''
  }
}
console.log(get(obj, 'a.b.replace')) 
console.log(get(obj, 'a.c.replace'))
```

