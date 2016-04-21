---
layout:     post
title:      "HTML&CSS 要点"
date:       2016-04-20
author:     "Sim"
catalog: false
tags:
    - HTML
    - CSS
---

1. 在新窗口打开链接

```html
<a href="http://google.com" target="_blank">Google</a>
```

2. 块级元素与内联元素(Block vs. Inline Elements)

块级元素在新的一行开始，可以嵌套在其他元素中，也可以包含内联元素。而内联元素不会另起新行，只包含了其内容的宽度。可以被嵌套在其他元素中，但是不能包含有块级元素。

3. css的`display`属性中，通常值是`block`,`inline`,`inline-block`和`none`.其中`inline-block`是将该元素作为块级元素，接收所有盒模型的属性。但是该元素仍会在跟其他元素在同一行中，不会另起新行。

4. 盒模型的宽度和高度计算

box-width = margin-right + border-right + padding-right + width + margin-left + border-left + padding-left
box-height = margin-top + border-top + padding-top + height + margin-bottom + border-bottom + padding-bottom

5. 内联元素没有`width`和`height`元素

6. css中常用的前缀：

  * Mozilla Firefox: -moz-
  * Microsoft Internet Explorer: -ms-
  * Webkit(Google Chrome and Apple Safari): -webkit-

7. Box Sizing的常见值:

  * content-box: 默认值。
  * padding-box: 将`padding`的属性值包含在`width`和`height`中。也就是说，使用`padding-box`的话，如果元素的宽是400，padding值是20的话，实际上该模型的宽仍然是400.
  * border-box: `border-box`同`padding-box`一样，不过就增加了`border`的值。就是说，元素的宽高包含了`border`和`padding`

8. `margin: 0 auto;代表对象上下间隔为0px，左右间隔根据对象宽度自适应。常用于让DIV布局居中。

9. 将`line-height`和`height`设置为相同值可以使文字垂直居中显示。

10. 字体属性的简写顺序: `font-style, font-variant, font-weight, font-size, line-height, font-family`

e.g

```css
html {
  font: italic small-caps bold 13px/22px "Helvetica Neue", Helvetica, Arial, sans-serif;
}
```

11. `box-shadow`属性用于向框添加一个或者多个阴影

语法: `box-shadow: h-shadow v-shadow blur spread color inset`

从左到右分别是：水平阴影位置，垂直阴影位置，模糊距离，阴影尺寸，阴影颜色，内部阴影

要注意的是，阴影尺寸的值是相对框的大小而言的。比方说当前div的width是150px, 阴影尺寸为-30px的话，实际上为120px。

12 `::before`用于在被选元素的内容前面插入内容，使用`content`来指定插入内容

13. css3的动画由`@keyframes`规则进行创建，在`@keyframes`中规定样式即可。

e.g

```css
@keyframes myAnimation {
  from { background: red; }
  to { background: yellow; }
}

/*使用*/
.myClass {
  animation: myAnimation 2s ease infinite;
}
```
