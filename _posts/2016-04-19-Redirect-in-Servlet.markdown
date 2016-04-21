---
layout:     post
title:      "Servlet中的重定向"
date:       2016-04-19
author:     "Sim"
catalog: false
tags:
    - Servlet
---

在Servlet中重定向请求到另一个页面，可以使用response对象的`sendRedirect()`方法。

该方法将响应和状态码以及新的页面发送到浏览器。也可以通过使用`setStatus()`和`setHeader()`来达到同样的请求

```Java
String site = "http://www.newpage.com";
response.setStatus(response.SC_MOVED_TEMPORARILY);
response.setHeader("Location", site);
```
