flex
容器的属性（父）
flex-direction row column \*-reverse
flex-wrap wrap nowrap wrap-reverse
flex-flow \<flex-direction> || \<flex-wrap>

    justify-content    flex-start | flex-end | center | space-between | space-around    默认横向主轴（若direction改变则不一定）
    align-items        flex-start | flex-end | center | baseline | stretch（item没高度或auto，将占满整个容器的高度）
    align-content      flex-start | flex-end | center | space-between | space-around | stretch   多根轴线的对齐方式 必须wrap

项目的属性（子）
order -1 0(defualt) 1 2 ... 项目的排列顺序
flex-grow default 0 放大比例 0 不放大 2 是 1 的一倍。。。 so on
flex-shrink default 1 空间不足，该项目将缩小
flex-basis default auto 原本大小 可以设置固定像素值
flex none | <flex-grow> <flex-shrink>? || <flex-basis>
flex: 0 1 auto; 缩短自身以适应 flex 容器
flex: 1 1 auto; 伸长并吸收 flex 容器中额外的自由空间
align-self auto | flex-start | flex-end | center | baseline | stretch
