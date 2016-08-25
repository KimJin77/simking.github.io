---
layout:     post
title:      "改变CSS中input的placeholder颜色"
date:       2016-08-24
author:     "Sim"
catalog: false
tags:
    - CSS
---

Stack Overflow上的答案:[Change an input's HTML5 placeholder color with CSS](http://stackoverflow.com/questions/2610497/change-an-inputs-html5-placeholder-color-with-css)

```css
::-webkit-input-placeholder { /* Webkit, Blink, Edge */
	color: black;
}

::-moz-placeholder { /* Mozila Firefox 19+  注意此处是双冒号*/
	color: black;
	opacity: 1;
}

:-moz-placeholder { /* Firefox 4 - 18, 注意此处是单冒号*/
	color: black;
}

:-ms-input-placeholder { /* Internet Explorer 10-11 */
	color: black;
}
```

