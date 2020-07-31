#### display

- 没有占位
- 引发回流
- 影响子元素

#### visibility

- 占位，不触发事件
- 重绘
- 子孙元素应用visibility:visible，可以显示
- transition不支持

#### opacity

- 占位，触发事件
- 不一定重绘
- 影响子元素
- 支持transition