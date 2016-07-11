---
layout:     post
title:      "CSS代码风格规范（转）"
date:       2016-07-06
author:     "Sim"
catalog: true
tags:
    - Coding Style
    - css
---

在Github上找到一份各种语言代码风格的集合帖，挑几个自己常用的语言翻译记录下来。方便查阅。

# CSS

## 文件名称

* 不要将项目的名称用在你的文件名上

```css
/* DON'T */
sales.css
checkout.css

/* DO */
base.css
component-name.css
```

* 将组件名作为样式表的名称

```css
/* DON'T */
styles.css
main.css
mobile.css
full-page.css
meli.css
sandra.css
a.css

/* DO */
payment-methods.css
message-boxes.css
categories.css
search-result.css
```

* 使用`base.css`作为基本文件的命名，比方说标题，段落等风格的设置。
* 使用'-'作为组件名称的分隔符号，而不是使用驼峰命名法或者是下划线

```
/* DON'T */
message_boxes.css
paymentMethods.css

/* DO */
message-boxes.css
payment-methods.css
```

* 可以通过在文件名后添加`.sufix`来对组件进行扩展。通常只会对特殊的组件进行扩展来满足需求

```
/* DO */
payment-methods.css
payment-methods.mla.css
```

* 双下划线可以作为样式表的修饰符。样式表的修饰符一般用来修改组件的风格。比方说，不同屏幕大小的组件风格不同：

```
/* DO */
payment-methods__large.css
payment-methods__small.css
payment-methods__ie8.css
```

## 常用格式

### 空间

* 使用4个空格作为缩进距离。不要使用tab
* 不要混合使用空格键和tab
* 移除掉每行末尾的空格
* 在选择器和大括号之间使用空格分隔
* 在属性和值之间使用空格进行分隔
* 在每个规则之间使用新行分隔

### 选择器 & 属性

* 每行只写一个选择器

```css
/* DO */
.selector1,
.selector2,
.selector3 {
	property: value;
}
```

* 每行只写一个属性

```css
/* DO */
.selector1 {
	property1: value;
	property2: value;
}
```

## 选择器

* 只用英文
* 使用连接符来分隔单词

```
/* DON'T */
.homePage
.box_registration

/* DO */
.home-page
.registration-box
```

* 在跨程序或者重用组件时使用app的前缀

```
/* DON'T */
.cho-header
.myml-sales

/* DO */
.commons-header
.ml-footer
```

* 尽可能地少使用ID选择器
* 避免使用标签选择器(tag)

## 属性

* 按照字母排列顺序进行声明
* 尽可能使用缩写

```
/* DON'T */
.selector {
	padding-bottom: 2em;
	padding-left: 1em;
	padding-right: 1em;
	padding-top: 0;
}

/* DO */
.selector {
	padding: 0 1em 2em;
}
```

* 某个值为0的时候不要添加单位

```
/* DON'T */
margin: 0em;

/* DO */
margin: 0;
```

* 值的大小在0-1之间时，省略掉0

```
/* DON'T */
margin: 0.5em;

/* DO */
margin: .5em;
```

* 使用单引号

```
/* DON'T */
background-image: url(sprite.png);
background-image: url("sprite.png);

/* DO */
background-image: url('sprite.png');
```

* 在表示颜色的时候，十六进制颜色用小写字母表示，可以使用缩写的话尽可能使用缩写

```
/* DON'T */
color: #111111;
background: #F3F3F3;

/* DO */
color: #111;
background: #f3f3f3;
```

* 设置背景颜色的时候使用rgba
* 记得每行代码背后加上';'
* 不要使用'!important'

## 注释

* 所有代码都应该可以被文档化
* 使用单行注释来添加提示，注意，建议事项或者警告
* 有需要的时候为代码添加注释

### 注释风格

* 文件头

```css
/**
 * Component Name
 * @authors: ...
 * @description: ...
 */
```

* 模块分隔

```css
/* Sidebar
---------------------------------------------------------------*/
```

* 行内注释

```css
.ch-price {
	line-height: 1em; /* 16px */
}

.form-actions {
	margin-left: 175px; /* label width + 15px */
}

.payment-methods {
	display: inline-block;
	height: 20px; /* default value, exceptions added to each logo */
	text-align: left; /* just in case the container has text-align: right */
}
```


