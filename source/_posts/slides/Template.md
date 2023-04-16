%标题
%作者
%日期

## 上述% 元信息说明

文本开头需要包含三行以%打头的元信息：这将生成 幻灯片入口：这些百分号 % 将单独生成幻灯片入口（切记上下行之间不能有空格）


## 级别说明 
- 如果是一级标题，紧接着的有内容和无内容时生成的ppt 样式是不一样的
- 如果所有的1级标题下无内容，则以二级标题为 分页依据，
且一级标题独立成页
- 有文字的时候，1级标题作为分页依据，其后内容作为内容


## 一级标题，紧接着的无内容举例:

    ```
  # level1 A
  ## leve2 A1
  - listA11
  - listA12
  ## leve2 A2
  - listA21
  - listA22
  # level1 B
  ## level2 B
  - list b1
  - list b2
    ```

上面会生成5页ppt，每个标题级分别占一页，其中前三个一组，竖向分页，后两个一组竖向分页， # level1 A 和# level1 B 横向划分

## 一级标题，紧接着的有内容举例:

    ```
  # level1 A
  - list l1a1
  - list l1a2
  ## leve2 A1
  - listA11
  - listA12
  ## leve2 A2
  - listA21
  - listA22
  # level1 B
  - list l1b1
  ## level2 B
  - list b1
  - list b2

    ```

上面会生成2页ppt， # level1 A 和# level1 B 横向划分,分别作为一页ppt，其后面的作为内容


## 这是一个幻灯片级别的标题 {data-background-color="#ff0000"}
- 幻灯片级别标题后 {data-background-color="#ff0000"} 可修改当前页背景
- 类似 的还有：
    * data-background-image
    * data-background-video
    * data-background-iframe


## 暂停

 `...` 可以暂停显示，这里是暂停之前

. . .

`...` 可以暂停显示，这里是暂停之后

## 强制分页
- `---` 可以强制分页  
- 这里是`---`强制分页前  

---

这里是`---`强制分页后

## 备注
- 这里是备注，演讲者模式（按S进入演讲者模式）时可以显示

::: notes
这里是备注，演讲者模式（按S进入演讲者模式）时可以显示


:::

## 列表逐行显示，按空格显示

::: incremental

- 列表项1
- 列表项2

:::

## 水平展示列表

:::::::::::::: {.columns}
::: {.column width="40%"}
第一列
:::
::: {.column width="60%"}
第2列
:::
::::::::::::::

## END