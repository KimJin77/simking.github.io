---
layout:     post
title:      "jQuery技巧"
subtitle:   "摘自《锋利的jQuery》"
date:       2016-04-13
catalog: true
tags:
    - jQuery
---

记录书上的一些jQuery技巧，由于书本和jQuery版本比较旧，所以可能存在用不了的情况。代码都还没尝试过。

### 1. 禁用页面的右键菜单

```JavaScript
$(document).ready(function() {
	$(document).bind("contextmenu", function(e) {
		return false;
	});
});
```

### 2. 新窗口打开页面

```JavaScript
$(document).ready(function() {
	// 1
	$('a[hreft^="http://"]').attr("target", "_blank");
	
	// 2 
	$("a[rel$='external']").click(function() {
		this.target = "_blank";
	});
});

// use
<a href="http://www.cssrain.cn" rel="external">Open link</a>
```

### 3. 判断浏览器类型

```JavaScript
$(document).ready(function() {
	// Firefox
	if ($.browser.mozilla && $.browser.version >= "1.8") {
	
	}
	
	// Safari
	if ( $.browser.safari ) {
	}
	
	// Chrome
	if ($.browser.chrome) {
	}
	
	// Opera
	if ($.browser.opera) {
	}
	
	// IE6以下
	if ($.browser.msie && $.browser.version <= 6) {
	}
	
	// IE6以上
	if ($.browser.msie && $.browser.version > 6) {
	}
});
```

1.3版本以后，官方推荐使用$.support来替代$.browser


### 4. 输入框文字焦点

```JavaScript
$(document).ready(function() {
	$("input.text1).val("Enter your search text here");
	textFill($("input.text1"));
});

function textFill(input) {
	var originalValue = input.val();
	input.focus( function() {
		if ($.trim(input.val()) == originalValue) {
			input.val("");
		}
	}).blur(funtion() {
		if ($.trim(input.val()) == "") {
			input.val(originalValue);
		}
	});
}
```

### 5. 返回头部滑动动画

```JavaScript
jQuery.fn.scrollTo = function(speed) {
	var targetOffset = $(this).offset().top;
	$("html.body").stop().animate({ scrollTop: targetOffset}, speed);
	return this;
};

// use
$("#goheader").click(function() {
	$("body").scrollTo(500);
	return false;
});
```

### 6. 获取鼠标位置

```JavaScript
$(document).ready(function() {
	$(document).mousemove(function(e) {
		$("#XY").html("X:" + e.pageX + " | Y:" + e.pageY);
	});
});
```

### 7. 判断元素是否存在

```JavaScript
$(document).ready(function() {
	if ($("#id").length) {
	
	}
});
```

### 8. 点击div跳转

```JavaScript
$("div").click(function() {
	window.location = $(this).find("a").attr("href");
	return false;
});
```

### 9. 根据浏览器大小添加不同样式

```JavaScript
$(document).ready(function() {
	function checkWindowSize() {
		if ($(window).width() > 1200) {
			$("body").addClass("large");
		} else {
			$("body").removeClass("large");
		}
	}
});
```

### 10.设置div在屏幕中央

```JavaScript
$(document).ready(function() {
	jQuery.fn.center = function() {
		this.css("position", "absolute");
		this.css("top", ($(window).height() - this.height()) / 2 + $(window).scrollTop() + "px");
		this.css("left", ($(window).width() - this.width()) / 2 + $(window).scrollLeft() + "px");
		return this;
	}
});

// use
$("#XY").center();
```

### 11. 关闭所有动画效果

```JavaScript
$(document).ready(function() {
	jQuery.fx.off = true;
});
```

### 12. 检测鼠标键

```JavaScript
$(document).ready(function() {
	$("#XY").mousedown(function(e) {
		alert(e.which); // 1 = 左键; 2 = 中键; 3 = 右键 
	});
});
```

### 13. 回车提交表单

```JavaScript
$(document).ready(function() {
	$("input").keyup(function(e) {
		if (e.which == "13") {
		 // Do something
		}
	});
});
```

### 14. 设置全局Ajax参数

```JavaScript
$("#load").ajaxStart(function() {
	showLoading();
	disableButtons();
});

$("#load").ajaxComplete(function() {
	hideLoading();
	enableButtons();
});
```

### 15. 获取选中的下拉框

```JavaScript
$("#someElement").find("option:selected");
$("#someElement option:selected");
```

### 16. 切换复选框

```JavaScript
var tog = false;
$("button").click(function() {
	$("input[type=checkbox]").attr("checked", !tog);
	tog = !tog;
});
```

### 17. 一段时间之后自动隐藏或关闭元素

```JavaScript
$("div").slideUp(300).delay(2000).fadeIn(400);
```

### 18. 限制textarea的字符个数

```JavaScript
jQuery.fn.maxLength = function(max) {
	this.each(function() {
		var type = this.tagName.toLowerCase();
		var inputType = this.type ? this.type.toLowerCase(): null;
		if (type == "input" && inputType == "text" || inputType == "password") {
			this.maxLength = max;
		} else if (type == "textarea") {
			this.onkeypress = function(e) {
				var ob = e || event;
				var keyCOde = ob.keyCode;
				var hasSelection = document.selection ? docuement.selection.createRange().text.length > 0 : this.selectionStart != this.selectionEnd;
				return !(this.value.length >= max && (keyCode > 50 || keyCode == 32 || keyCode == 0 || keyCode == 13) && !ob.crtlKey && !ob.altKey && !hasSelection);
			}
			this.onkeyup = function() {
				if (this.value.length > max) {
					this.value = this.value.substring(0, max);
				}
			};
		}
	});
};

// use
$("myTextarea").maxLength(10);
```

### 19. 扩展String对象方法

```JavaScript
$.extend(String.prototype, {
	isPositiveInteger: function() {
		return (new RegExp(/^[1-9]\d*$/).test(this));
	},
	
	isInteger: function() {
		return (new RegExp(/*\d+$/).test(this));
	},
	
	isNumber: function(value, element) {
		return (new RegExp(/^-?\d+|\d{1,3|(?:.\d{3})+)?$/).test(this));
	},
	
	trim: function() {
		return this.replace(/(^\s*)|(\s*$)|\r|\n/g, "");
	},
	
	trans: function() {
		return this.replace(/&lt;/g, '<').replace(/&gt;/g, '>').replace(/&quot;/g, '"');
	},
	
	replaceAll: function(os, ns) {
		return this.replace(new RegExp(os, "gm"), ns);
	},
	
	skipChar: function(ch) {
		if (!this || this.length === 0) { return ""; }
		if (this.charAt(0) === ch) { return this.substring(1).skipChar(ch); }
		return this;
	},
	
	isValidPwd: function() {
		return (new RegExp(/^[_]|[a-zA-Z0-9]){6,32}$/).test(this));
	},
	
	isValidMail: function() {
		return (new RegExp(/^\w+((-\w+))*\@[A-Za-z0-9]+((\.|-)[A-Za-z0-9]+)*\.[A-Za-z0-9]+$/).test(this.trim()));
	},
	
	isSpaces: function() {
		for (var i = 0; i < this.lenth(); i+=1) {
			var ch = this.charAt(i);
			if (ch!=" "&&ch!="\n"&&ch!="\t"&&ch!="\r") return false;
		}
		return true;
	},
	
	isPhone: function() {
		return (new RegExp(/(^[0-9]{3,4}[-])?\d{3,8}(-\d{1,6})?$)|(^\[0-9]{3,4}\)\d{3,8}(\(\d{1,6}\))?$)|(^\d{3,8}$)/).test(this));
	},
	
	isUrl: function() {
		return (new RegEx[(\^[A-Za-z]+:\/\/([a-zA-Z0-9\-\.\+)([-\w.\/?%&=:]*)$/).test(this));
	}
	
	isExternalUrl: function() {
		return this.Url() && this.indexOf("://" + document.domain) == -1;
	}
});
```