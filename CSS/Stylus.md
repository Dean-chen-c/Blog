#### Stylus特点

- 选择器

  - ```stylus
    body,html
      color: white
      &:hover
        color: black
    ```

- 变量

  - ```stylus
     #logo
       position: absolute
       top: 50%
       left: 50%
       width: 150px
       height: 80px
       margin-left: -(@width / 2)
       margin-top: -(@height / 2)
    ```

- 循环

  - ```stylus
    table
      for row in 1 2 3 4 5
        tr:nth-child({row})
          height: 10px * row
    ```

- 

