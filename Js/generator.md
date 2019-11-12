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
run(function*() {
  console.log(2);
  const uri = "http://47.112.101.19:9990/sw-cms/api/queryAllChannel/cn";
  console.log(3);
  const res = yield fetch(uri);
  console.log(res);
  const post = yield res.json();
  console.log(post);
});
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
