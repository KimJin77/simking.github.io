---
layout:     post
title:      "jquery 滚动到某个子元素"
date:       2016-08-25
author:     "Sim"
catalog: false
tags:
    - JavaScript
---

项目需求做了个城市列表的选择，在添加索引栏的时候折腾了好久。原因在于元素的滚动上。

上网看了挺多帖子，元素滚动大多数是使用`$('html, body').animate({ scrollTop: 1000 }, 1000);`这样的方法来完成。但是，我按照一模一样的做法，点击元素栏的时候，却纹丝不动。之后一直变着法子试，`scrollTop()`都用烂了还是没出现效果。

最后发现，其实上面的方法是没有错的，只是我需要的滚动是body子元素div子元素的滚动。滚动条是在div里面，而不是在body上面，body的高度只是刚好等于浏览器的大小，所以就一动也不动了。

最后根据[http://www.angelweb.cn/Html/jquery/jqueryjiaocheng/2957.html](http://www.angelweb.cn/Html/jquery/jqueryjiaocheng/2957.html)的做法，完成了子元素的滚动到顶部的效果。

我自己的代码如下：

```js
$('#indexBar').on('click', 'li', function(){
			var character = $(this).data('value');
			var offsetTop = $('div[data-group="' + character + '"]').offset().top;
			var navBarHeight = $('.weui_navbar').height();
			var marginTop = parseFloat($('div[data-group="' + character + '"]').css('margin-top'));

			$('#cityList').scrollTop(offsetTop - $('#cityList').offset().top + $('#cityList').scrollTop() - navBarHeight - marginTop);	
		});
```

因为我的界面是有个导航栏，所以需要减去导航栏的高度，另外，因为margin的距离，会出现滚动出现些许偏差，最后也减去了。

---

这样的里面不知道对不对

** `offset().top`和`scrollTop()`的区别

* `offset().top`是指当前元素距离body的顶部距离
* `scrollTop()`是指滚动消失掉的距离


