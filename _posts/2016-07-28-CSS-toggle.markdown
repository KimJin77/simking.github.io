---
layout:     post
title:      "CSS中toggle用法"
date:       2016-07-28
author:     "Sim"
catalog: false
tags:
    - css
---

目前项目中有个需求是点击某个element，然后其详细内容会根据当前的状态隐藏或者显示。这种需求在手机app端的话，得写好几段代码才能解决。于是想起css中似乎有个`trigger()`函数可以完成。不过尝试了一下，发现`trigger()`似乎是用来触发其他方法，没办法完成隐藏或显示。
最后发现了`toggle()`函数。`toggle()`会根据元素当前的状态进行显示或者隐藏，还可以为动画设置速率。不过需要注意的一点是, `toggle()`函数在显示时，元素的样式是`inline-block`，也就是说触发显示状态的时候，该元素`display: inline-block`。一开始没去注意，调了很久的样式，后来才发现了这一点。如果想要其显示为block的话，可以这样写:

```css
$('.className').toggle('medium', function() {
	if ($(this).is(':visible')) {
		$(this).css('display', 'block');
	}
});
```

其中`medium`是速率，具体的其他参数，可以参考W3School的介绍


