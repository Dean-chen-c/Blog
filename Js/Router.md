# Router

### hashRouter

hash 路由一个明显的标志是带有#,我们主要是通过监听 url 中的 hash 变化来进行路由跳转。

```javascript
class Routers {
  constructor() {
    // 储存hash与callback键值对
    this.routes = {};
    // 当前hash
    this.currentUrl = "";
    // 记录出现过的hash
    this.history = [];
    // 作为指针,默认指向this.history的末尾,根据后退前进指向history中不同的hash
    this.currentIndex = this.history.length - 1;
    this.refresh = this.refresh.bind(this);
    this.backOff = this.backOff.bind(this);
    // 默认不是后退操作
    this.isBack = false;
    window.addEventListener("load", this.refresh, false);
    window.addEventListener("hashchange", this.refresh, false);
  }

  route(path, callback) {
    this.routes[path] = callback || function() {};
  }

  refresh() {
    this.currentUrl = location.hash.slice(1) || "/";
    if (!this.isBack) {
      // 如果不是后退操作,且当前指针小于数组总长度,直接截取指针之前的部分储存下来
      // 此操作来避免当点击后退按钮之后,再进行正常跳转,指针会停留在原地,而数组添加新hash路由
      // 避免再次造成指针的不匹配,我们直接截取指针之前的数组
      // 此操作同时与浏览器自带后退功能的行为保持一致
      if (this.currentIndex < this.history.length - 1)
        this.history = this.history.slice(0, this.currentIndex + 1);
      this.history.push(this.currentUrl);
      this.currentIndex++;
    }
    this.routes[this.currentUrl]();
    console.log("指针:", this.currentIndex, "history:", this.history);
    this.isBack = false;
  }
  // 后退功能
  backOff() {
    // 后退操作设置为true
    this.isBack = true;
    this.currentIndex <= 0
      ? (this.currentIndex = 0)
      : (this.currentIndex = this.currentIndex - 1);
    location.hash = `#${this.history[this.currentIndex]}`;
    this.routes[this.history[this.currentIndex]]();
  }
}
```

```javascript
window.Router = new Routers();
const content = document.querySelector("body");
const button = document.querySelector("button");
function changeBgColor(color) {
  content.style.backgroundColor = color;
}

Router.route("/", function() {
  changeBgColor("yellow");
});
Router.route("/blue", function() {
  changeBgColor("blue");
});
Router.route("/green", function() {
  changeBgColor("green");
});

button.addEventListener("click", Router.backOff, false);
```

```html
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>hash router</title>
  </head>
  <body>
    <ul>
      <li><a href="#/">turn yellow</a></li>
      <li><a href="#/blue">turn blue</a></li>
      <li><a href="#/green">turn green</a></li>
    </ul>
    <button>back</button>
    <script src="./hash.js" charset="utf-8"></script>
  </body>
</html>
```

### history api

浏览器中查询出 History API

```javascript
window.history.back(); // 后退
window.history.forward(); // 前进
window.history.go(-3); // 后退三个页面
```

- history.pushState 用于在浏览历史中添加历史记录,但是并不触发跳转
  > _state_:一个与指定网址相关的状态对象，popstate 事件触发时，该对象会传入回调函数。如果不需要这个对象，此处可以填 null。
  > _title_：新页面的标题，但是所有浏览器目前都忽略这个值，因此这里可以填 null。
  > _url_：新的网址，必须与当前页面处在同一个域。浏览器的地址栏将显示这个网址。
- history.replaceState 修改浏览历史中当前纪录,而非添加记录,同样不触发跳转

- popstate 每当同一个文档的浏览历史（即 history 对象）出现变化时，就会触发 popstate 事件。
  > 需要注意的是，仅仅调用 pushState 方法或 replaceState 方法 ，并不会触发该事件，只有用户点击浏览器倒退按钮和前进按钮，或者使用 JavaScript 调用 back、forward、go 方法时才会触发。

```javascript
class Routers {
  constructor() {
    this.routes = {};
    // 在初始化时监听popstate事件
    this._bindPopState();
  }
  // 初始化路由
  init(path) {
    history.replaceState({ path: path }, null, path);
    this.routes[path] && this.routes[path]();
  }
  // 将路径和对应回调函数加入hashMap储存
  route(path, callback) {
    this.routes[path] = callback || function() {};
  }

  // 触发路由对应回调
  go(path) {
    history.pushState({ path: path }, null, path);
    this.routes[path] && this.routes[path]();
  }
  // 监听popstate事件
  _bindPopState() {
    window.addEventListener("popstate", e => {
      const path = e.state && e.state.path;
      this.routes[path] && this.routes[path]();
    });
  }
}
```

```javascript
window.Router = new Routers();
Router.init(location.pathname);
const content = document.querySelector("body");
const ul = document.querySelector("ul");
function changeBgColor(color) {
  content.style.backgroundColor = color;
}

Router.route("/", function() {
  changeBgColor("yellow");
});
Router.route("/blue", function() {
  changeBgColor("blue");
});
Router.route("/green", function() {
  changeBgColor("green");
});

ul.addEventListener("click", e => {
  if (e.target.tagName === "A") {
    e.preventDefault();
    Router.go(e.target.getAttribute("href"));
  }
});
```

```html
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>h5 router</title>
  </head>
  <body>
    <ul>
      <li><a href="/">turn yellow</a></li>
      <li><a href="/blue">turn blue</a></li>
      <li><a href="/green">turn green</a></li>
    </ul>
  </body>
</html>
```
