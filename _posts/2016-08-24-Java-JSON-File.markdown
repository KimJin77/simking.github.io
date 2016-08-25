---
layout:     post
title:      "Java读写JSON文件"
date:       2016-08-24
author:     "Sim"
catalog: false
tags:
    - Java
    - JSON
---

### 从文件中读取json

```java
String str = "";
		BufferedReader reader = null;
		try {
			FileInputStream fileInputStream = new FileInputStream("filePath");
			InputStreamReader inputStreamReader = new InputStreamReader(fileInputStream, "utf-8");
			String tempStr = null;
			reader = new BufferedReader(inputStreamReader);
			while((tempStr = reader.readLine()) != null) {
				str += tempStr;
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
		JSONObject json = JSONObject.fromObject(str);
		System.out.println(json);
```

### 将json写入到json文件

原文地址:[http://zhihaozhang.github.io/2016/03/17/用Java生成Json文件/](http://zhihaozhang.github.io/2016/03/17/用Java生成Json文件/)

```java
public void writeToJson(String filePath,JSONArray object) throws IOException
	{
	    File file = new File(filePath);
	    char [] stack = new char[1024];
	    int top=-1;
	
	    String string = object.toString();
	
	    StringBuffer sb = new StringBuffer();
	    char [] charArray = string.toCharArray();
	    for(int i=0;i<charArray.length;i++){
	        char c= charArray[i];
	        if ('{' == c || '[' == c) {  
	            stack[++top] = c; 
	            sb.append("\n"+charArray[i] + "\n");  
	            for (int j = 0; j <= top; j++) {  
	                sb.append("\t");  
	            }  
	            continue;  
	        }
	         if ((i + 1) <= (charArray.length - 1)) {  
	                char d = charArray[i+1];  
	                if ('}' == d || ']' == d) {  
	                    top--; 
	                    sb.append(charArray[i] + "\n");  
	                    for (int j = 0; j <= top; j++) {  
	                        sb.append("\t");  
	                    }  
	                    continue;  
	                }  
	            }  
	            if (',' == c) {  
	                sb.append(charArray[i] + "");  
	                for (int j = 0; j <= top; j++) {  
	                    sb.append("");  
	                }  
	                continue;  
	            }  
	            sb.append(c);  
	        }  
	        Writer write = new FileWriter(file);  
	        write.write(sb.toString());  
	        write.flush();  
	        write.close();  
	}
```

