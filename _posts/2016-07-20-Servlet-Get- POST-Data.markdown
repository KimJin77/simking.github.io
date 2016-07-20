---
layout:     post
title:      "Servlet获取POST过来的数据"
date:       2016-07-20
author:     "Sim"

catalog: false
tags:
    - Servlet
    - Java
---

通常情况下，在Servlet的doGet()和doPost()的方法中，都可以通过'req.getParameter("key")'来获取相对应的变量。不过这种通常的情况对于POST的数据来说，仅适用于content-type为'application/x-www-form-urlencoded'的时候。

如果对post的数据使用了其他编码方式的话，在doPost()方法中用'req.getParameter("key")'读取出来的变量将为null。通常来说我们post给Server的数据都会是json格式，读取json格式的话，我们需要一个解码器来进行解析：`BufferedReader reader = request.getReader();'

e.g:

```java
public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	StringBuffer buffer = new StringBuffer();
	String line = null;
	try {
		BufferedReader reader = request.getReader();
		while((line = reader.readLine()) != null) {
			buffer.append(line);
		}
	} catch (Exception e) {
		e.printStackTrace();
	}
	
	try {
		JSONObject jsonObject = JSONObject.fromObject(buffer.toString());
	} catch (ParseException e) {
		throw new IOException("Error parsing JSON request string");
	}
}
```


