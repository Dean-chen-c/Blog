#### flex布局

```scss
.p{
  display:flex;
  background:#ccc;
  // flex-wrap:wrap;
  // flex-direction:column;
  justify-content:space-around;
  align-items:flex-end;
  // align-content:center;
  height:50px;
  .c{
    flex-basis:400px;
    flex-grow:0;
    flex-shrink:0;
    text-align:center;
    background:#ceb0b0;
  }
  .c1{
    order:5;
   flex-shrink:1;
   flex-basis:100px;
  }
  .c5{
    order:-1;
  }
  .c3{
    align-self:flex-start;
  }
}


```

```html
<div class='p'>
  <div class='c c1'>1</div>
  <div class='c c2'>2</div>
  <div class='c c3'>3</div>
  <div class='c c4'>4</div>
  <div class='c c5'>5</div>
</div>
```

