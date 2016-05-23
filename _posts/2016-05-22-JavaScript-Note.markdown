---
layout:     post
title:      "JavaScript笔记"
date:       2016-05-22
author:     "Sim"
catalog: false
tags:
    - JavaScript
---

1. JS中变量有按值和按引用两种访问方式，其中基本类型都是按值访问。但是对于方法的参数来说，只有按值传递这种方式。

```javascript
//按值传递
var num1 = 5;
var num2 = num1;

// num2与num1中的值完全独立，修改num2并不会影响到num1

// 按引用传递
var obj = new Person();
var obj2 = obj;
obj.name = "Tim";
alert(obj2.name);

// 此时会输出Tim，因为Object是按引用进行访问的。
```

2. 使用`typeof`操作符可以检测某个对象是否为基本数据类型。但是如果想要检测引用类型的值时，这个操作符的作用就没那么大了。通常我们使用`instanceof`操作符来检测某个值是哪种类型的对象。

```javascript
result = variable instanceof constructor
```

3. 创建Object实例的两种方法

  1）new操作符+Object构造函数

  ```javascript
  var person = new Object();
  person.name = "Tom";
  person.age = 13;
  ```

  2) 对象字面量表示法

  ```javascript
  var person = {
    name: "Tom",
    age : 29
  };
  ```

4. JavaScript的数组中的length属性不是只读的！所以可以通过调整length的值来进行增删

```javascript
// 移除“blue”
var colors = ["red", "orange", "blue"];
colors.length = 2;
```

5. 函数中的`arguments`有一个`callee`属性，该属性是一个指针，指向拥有这个arguments对象的函数

```javascript
function factorial(num) {
  if (num < 1) return 1;
  else return num * arguments.callee(num-1);
}
```
