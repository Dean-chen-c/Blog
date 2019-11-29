## IntersectionObserver

**IntersectionObserver** 翻译为 "交叉观察者"，它的任务就是监听目**标元素**跟指定**父元素**(只能是父类元素)（用户可指定，默认为 viewport）是否在发生交叉行为

### 使用方法

#### 构造函数

```javascript
new IntersectionObserver(callback, options);
```

#### callback

```javascript
new IntersectionObserver(entries => {
  entries.forEach(item => console.log(item));
  /*
    item里面包含哪些常用属性:
    1、boundingClientRect 空间信息
    2、intersectionRatio 元素可见区域的占比
    3、isIntersecting 元素是否可见
    4、target 	目标节点
  */
});
```

> 页面初始化的时候会触发一次 callback，entries 为所有已监听的目标集合

#### options

| 属性       | 说明                                                                                  |
| ---------- | ------------------------------------------------------------------------------------- |
| root       | 指定父元素，默认为视窗                                                                |
| rootMargin | 触发交叉的偏移值，默认为"0px 0px 0px 0px"（上左下右，正数为向外扩散，负数则向内收缩） |

#### 常用方法

| 名称        | 说明                       |
| ----------- | -------------------------- |
| observe     | 监听一个目标元素           |
| unobserve   | 停止监听一个目标元素       |
| takeRecords | 返回所有监听的目标元素集合 |
| disconnect  | 停止所有监听               |

#### 例子（是否在可见）

```javascript
const callback = (entries, observer) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      //doingsomethring
      // 取消监测
      observer.unobserve(document.getElementById("target"));
    }
  });
};
const observer = new IntersectionObserver(callback);
observer.observe(document.getElementById("target"));
```

#### 应用

1. 图片懒加载
   传统的懒加载只是监听全局滚动条的滚动，像这种小细节还是无法实现的（传统的实现方法并不是判断目标是否出现在视窗，所以横向的图片会一起加载，即使你没有向左滑动）
2. 触底
   滚动到了底部，开始请求数据
3. 吸顶
   position: sticky

```html
<div class="p">
  <div class="o"></div>
  <!-- 参照元素 -->
  <div class="reference"></div>

  <nav>我可以吸顶</nav>
</div>
```

```scss
.p {
  height: 2000px;
}
.o {
  height: 200px;
}
nav {
  background: #ccc;
  &.fixed {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
  }
}
```

```javascript
let nav = document.querySelector("nav");
let reference = document.querySelector(".reference");
let item;
new IntersectionObserver(entries => {
  item = entries[0];

  console.log(item.target.getBoundingClientRect());
  let top = item.boundingClientRect.top;
  console.log(top);

  // 当参照元素的的top值小于0，也就是在视窗的顶部的时候，开始吸顶，否则移除吸顶
  if (top < 0) nav.classList.add("fixed");
  else nav.classList.remove("fixed");
}).observe(reference);
```
