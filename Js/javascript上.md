# 作用域

### js代码编译

- 代码分词
  - 过程中会生成**词法作用域**
- 代码解析
  - 过程中生成AST（抽象语法树）
- 代码生成
  - 过程中会出现查询LHS（赋值）、RHS（取值）

### 词法作用域

应该是在写代码的时候就确定了

> ⚠️存在欺骗词法
>
> 就是在代码运行时来修改词法作用域
>
> eval函数

```js
var b = 2;
function foo(str,a){
  eval(str);
  console.log(a,b)
}
foo('var b = 3;',1)
// 1 3
```

### 函数作用域

函数外部无法访问函数内部声明的变量，可以达到**“隐藏”变量的目的**（私有化）

##### “隐藏”变量的好处

- 私有化变量
- 规避冲突
- 可以进行模块管理

##### 函数

- 匿名函数
- 具名函数（还是会污染全局作用域）

##### 函数表达式(立即执行)

```js
(function foo(){
  ...
})();
  
(function (global){
  ...
})(window)

(function (undefined){
  //规避undefined被赋值
}())
```

### 块级作用域

{}为块

```js
try{
  undefined();
}catch(err){
  console.log(err);
}
console.log(err); //ReferenceError
```

let

const

#### 提升

代码编译的时候**声明**，运行的时候**赋值**

```js
foo() //TypeError
bar() //RefrenceError
var foo= function bar(){
	//...
}
```

函数优先（先函数后变量）

```js
foo();
var foo= function(){
  console.log(1);
}
function foo(){
  console.log(2)
}
foo();

//2 1
```

#### 闭包

基于词法作用域书写代码时所产生的自然结果

词法作用域是闭包的一部分，最重要的是闭包可以**保留对作用域的引用**（不会被垃圾回收）

可以解释为：通过某种手段将内部函数传递到所在词法作用域以外，它都会持有对原始定义作用域的引用。

#### 模块

```js
var modules=(function(){
  var modules={};
  function define(name,deps,impl){
    for(let i=0;i<deps.length;i++){
      deps[i]=modules[deps[i]]
    }
    // this 指向impl 本身
    modules[name]=impl.apply(impl,deps);
  }
  function get(name){
    return modules[name];
  }
  
  return {
    define:define,
    get:get
  }
})();
 modules.define('bar',[],function(){
   function hello(who){
     return 'hello'+who;
   }
   return {
     hello:hello
   }
 })
var bar =modules.get('bar');
bar.hello()

```

#### 动态作用域

js没有动态作用域，但是this机制很像动态作用域

```js

function bar(){
  var a=3;
  function foo(){
  console.log(a)
}
  return foo
}
var a=2;
bar();
```

# This

#### this指向

错误理解：

- 指向函数本身
- 指向函数作用域

**this是在运行时绑定的，只取决于函数的调用方式**（执行上下文、调用方式、传入参数）

#### 默认绑定

```js
function foo(){
	console.log(this.a); //this => window
}
foo();
```

#### 隐式绑定

```js
var obj={
  a:1,
  foo:foo
};
function foo(){
	console.log(this.a); //this => obj
}
obj.foo();
//对象引入链，只在最后一层起作用
//obj3.obj2.obj1.foo() this=> obj1
```

**隐式丢失**

```js
// f 是 obj.foo的引用 实际上他是引用foo的本身 所以采用了 默认绑定
var obj={
  a:1,
  foo:foo
};
function foo(){
	console.log(this.a); //this => window
}
var f=obj.foo;
f();
```

```js
//隐式赋值 fn=obj.foo 类似
function foo(){
	console.log(this.a); //this => window
}
function doFoo(fn){
  fn();
}
var obj={
  a:1,
  foo:foo
};
doFoo(obj.foo);
setTimeout(obj.foo,100);
```

#### 显式绑定

- call

- apply

- bind

#### new绑定

```js
function foo(num){
	this.a = num;
}
var obj={};
var bar = foo.bind(obj);
bar(2);
console.log(obj.a)

var bar2 = new bar(3)
console.log(obj.a)
console.log(bar2.a)
```

#### 优先级

new > 显式 > 隐式 > 默认

#### 绑定例外

把**null**，**undefined**传入call apply bind 的时候，应用默认绑定

```js 
function foo(a,b){
  console.log('a:' + a + ',b:' + b)
}
var ø = Object.create(null);

foo.apply(ø,[2,3]);

var bar = foo.bind(ø,2);
bar(3);
```

```js
function foo(){
  console.log(this.a)
}
var a = 2;
var o = {a:3,foo:foo};
var p = {a:4};
(p.foo = o.foo)()
```

#### 软绑定

```js
if(!Funtion.prototype.softBind){
  Function.prototype.softBind = function(obj){
    var fn = this;
    var curried = [].slice.call(arguments, 1);
    var bound = function(){
      return fn.apply(
        (!this||this === (window||global))? obj:this,
         [].concat.apply(curried,arguments)
      );
      bound.prototype = Object.create(fn.prototype);
      return bound;
    }
  }
}

function foo(){
    console.log("name: "+this.name);
}

var obj1={name:"obj1"},
    obj2={name:"obj2"},
    obj3={name:"obj3"};

var fooOBJ=foo.softBind(obj1);
fooOBJ();//"name: obj1" 在这里软绑定生效了，成功修改了this的指向，将this绑定到了obj1上

obj2.foo=foo.softBind(obj1);
obj2.foo();//"name: obj2" 在这里软绑定的this指向成功被隐式绑定修改了，绑定到了obj2上

fooOBJ.call(obj3);//"name: obj3" 在这里软绑定的this指向成功被硬绑定修改了，绑定到了obj3上

setTimeout(obj2.foo,1000);//"name: obj1"
/*回调函数相当于一个隐式的传参，如果没有软绑定的话，这里将会应用默认绑定将this绑定到全局环
境上，但有软绑定，这里this还是指向obj1*/


var a=2;
function foo(){
}
foo.a=3;
Function.prototype.softBind=function(){
    var fn=this;
    return function(){
        console.log(fn.a);
    }
};
Function.prototype.a=4;
Function.prototype.softBind.a=5;

foo.softBind()();//3
Function.prototype.softBind()();//4
```

#### 箭头函数

```js
function foo(){
  return ()=>{
    console.log(this.a)
  }
}
var obj1={a:2}
var obj2={a:3}
var bar = foo.call(obj1)
bar.call(obj2)
```

