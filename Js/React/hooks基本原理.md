```js
const R = (function () {
  let _val;
  return {
    render(C) {
      const _C = C();
      _C.render();
      return _C;
    },
    u(i) {
      _val = _val || i;
      // console.log('i',i)
      function set(n) {
        // console.log('n',n)
        _val = n;
      }
      return [_val, set];
    },
    o() {
      console.log(_val);
    },
  };
})();

function Counter() {
  const [c, setC] = R.u(0);
  return {
    click: () => setC(c + 1),
    render: () => console.log("r", c),
  };
}

// const [f,setF]=R.u(0)
// console.log(f)
// setF(1)
// console.log(f)

let App;
App = R.render(Counter);
App.click();
App = R.render(Counter);
```

