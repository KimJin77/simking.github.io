---
layout:     post
title:      "JS获取日期 & bootstrap-datepicker设置默认日期"
date:       2016-07-11
author:     "Sim"
catalog: true
tags:
    - JavaScript
    - Bootstrap-datepicker
---

# JavaScript获取日期

```javascript
function getDateString(dayInterval) {
	var dd = new Date();
	dd.setDate(dd.getDate() + dayInterval);
	var y = dd.getFullYear();
	var m = dd.getMonth() + 1; // month starts from 0
	var d = dd.getDate();
	return y + "-" + m + "-" + d;
}
```

e.g. 

```javascript
// Yesterday
var yesterday = getDateString(-1);
// Today
var today = getDateString(0);
```

# Bootstrap-datepicker设置默认日期

```css
<input type="text" class="form-control" id="start_date">
```

```javascript
// warning: date format should be consistent
$("#start_date").datepicker({
	// set format and so on
});

$("#start_date").datepicker("setDate", new Date());


```

