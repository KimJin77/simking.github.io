---
layout:     post
title:      "JS获取手机操作系统"
date:       2016-07-29
author:     "Sim"
catalog: false
tags:
    - JavaScript
---

1. 获取手机系统版本

```js
/**
 * Get operator system 
 *
 * @return {String} system
 */
function getOS() {
	var ua = navigator.userAgent;
	if (ua.indexOf("Windows NT 5.1") != -1) return "Windows XP";
	if (ua.indexOf("Windows NT 6.0") != -1) return "Windows Vista";
	if (ua.indexOf("Windows NT 6.1") != -1) return "Windows 7";
	if (ua.indexOf("iPhone") != -1) return "iPhone";
	if (ua.indexOf("iPad") != -1) return "iPad";
	if (ua.indexOf("Linux") != -1) {
		var index = ua.indexOf("Android");
		if (index != -1) {
			//os以及版本
			var os = ua.slice(index, index+13);

			//手机型号
			var index1 = ua.lastIndexOf(";");
			var index2 = ua.indexOf("Build");
			var type = ua.slice(index1+1, index2);
			isAndroid = true;
			return type + os;
		} else {
			return "Linux";
		}
	}
	return "未知操作系统";
}
```

2. 利用js获取当前微信版本号

```js
var wx = navigator.userAgent.match(/MicroMessenger\/([\d\.]+)/i);
var version = wx[1]; // 微信版本
```


