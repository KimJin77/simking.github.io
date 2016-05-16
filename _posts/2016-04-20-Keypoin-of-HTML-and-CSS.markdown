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

## Update in 2016-05-10

1. 在表单中，如果在表单提交时不允许某个数据为空，则可以在`input`标签中添加`required`字段，表明该值不能为空

e.g

```html
<input type="text" required>...</input>
```

需要注意的是，`required`是HTML5添加的属性，在Safari中并不会生效

2. CSS属性修改的优先级：`!important` > 内部style声明 > id > class

3. Bootstrap

  * 图片适应屏幕： `img-responsive`
  * 文字居中：`text-center`
  * 创建Bootstrap按钮：`class="btn"`
  * 创建block的按钮：`class="btn btn-block"`

---

## Update in 2016-05-09

1. 浮动导致的布局变动

  * 对于块级元素来说，在不设置宽度的情况下，默认宽度是100%，一旦设置了浮动，它的宽度就会根据内容进行自动调整。
  * 设置了浮动的元素会脱离正常的文档流。譬如：默认情况下，父元素的高度会根据子元素的内容自动进行调整，而如果我们将子元素设置为浮动，父元素的高度就会变成0
  * 浮动的元素虽然脱离了文档流，但是里面的内容仍然会占据空间，会根据相对位置进行布局

2. CSS中的position（元素的定位属性），有5个可选值: static（默认值）、relative、absolute、fixed、inherit。其中static和relative用于相对定位。absolute和fixed属于绝对定位的范畴。

absolute是相对上一个**不为static的父元素**进行绝对定位。fixed则是相对于浏览器窗口进行定位。

3. 实际项目中，标签选择器一般用于定义全局样式

4. 使用个性化字体的话，使用`@font-face`，它可以加载服务器端的字体文件。基本语法如下

```CSS
@font-face {
  font-family: myFirstFont;
  src: url('Sansation_light.ttf');
}
```

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
  * Opera: -o-

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

13. `margin`指的是控件边缘相对父控件的边距，即自身边框到另一个容器边框之间的距离（容器外距离）。`padding`指的是控件内容相对控件边缘的距离，即自身边框到自身内部另一个容器边框的距离（容器内距离）。

14. css中`background`设置时，`background-size`不能在简写形式中赋值。另外，`background-size`有四种类型的值，分别是:

  * length: 设置背景图像的宽高。如果只设置一个值得话，另外一个的值是auto
  * percentage: 以父元素的百分比来设置背景图像的宽高。如果只设置一个值得话，另外一个的值是auto
  * cover: 把背景图像设置至足够大，以使其能够完全覆盖背景区域。
  * contain: 使宽高适应内容区域。

15. jQuery的尺寸方法

  * width()和height(): 设置或返回元素的宽高（不包含内边距，外边距或边框）
  * innerWidth()和innerHeight(): 返回元素的宽高（包括内边距）
  * outerWidth()和outerHeight(): 返回元素的宽高（包括内边距和边框）

16. css的绝对定位是“相对于”最近的已定位的祖先元素，而相对定位是“相对于”元素在文档中初始位置。
