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

6. ECMAScript中有两个属性：数据属性和访问器属性

  1) 数据属性：

    * [[Configurable]]: 表示能否通过delete删除属性从而定义属性，能否修改属性的特性，或者能够把属性修改为访问器属性。默认值为true

    * [[Enumerable]]: 表示能否通过for-in循环返回属性。默认值为true

    * [[Writable]]: 表示能否修改属性的值。默认值为true

    * [[Value]]: 包含这个属性的数据值。读取属性值时，从这个位置读；写入属性值时，把新值保存到这个位置。默认是undefined


  要修改属性默认的特性，必须使用`Object.defineProperty()`方法。这个方法接收三个参数：属性所在的对象，属性的名字和一个描述符对象。

  ```javascript
  var person = {};
  Object.defineProperty(person, "name") {
    writable: false,
    value: "Nicholas"
  }

  alert(person.name); // "Nicholas"
  person.name = "Greg";
  alert(person.name); // "Nicholas"
  ```

  可以多次调用`Object.defineProperty()`方法修改同一个属性，但在把configurable设置为false之后就会有限制。并且在调用时，如果没指定的话，默认值都是false。

  2）访问器属性

  * [[Configurable]]: 表示能否通过delete删除属性从而定义属性，能否修改属性的特性，或者能够把属性修改为访问器属性。默认值为true

  * [[Enumerable]]: 表示能否通过for-in循环返回属性。默认值为true

  * [[Get]]: 在读取属性时调用的函数，默认值是false

  * [[Set]]: 在写入属性时调用的函数，默认值是false

  访问器属性不能直接定义，必须使用`Object.defineProperty()`

  ```javascript
  var book = {
    _year: 2016,
    edition: 1
  };

  Object.defineProperty(book, "year", {
    get: function() {
      return this._year;
    }

    set: function(newValue) {
      if (newValue > 2015) {
        this._year = newValue;
        this.edition += newValue - 2015;
      }
    }
  });

  book.year = 2016;
  alert(book.edition); // 2
  ```

7. 使用`Object.getOwnPropertyDescriptor()`方法获取给定属性的描述符。该方法接收属性所在的对象和属性名称两个参数。如果是访问器属性的话，是configurable, enumerable, get和set。如果是数据属性的话，则是configurable, enumerable, writable和Value

```javascript
var book = {};
Object.definePropertries(book, {
  _year: {
    value: 2016
  },

  edition: {
    value: 1
  },

  year: {
    get: function() {
      return this._year;
    },

    set: function(newValue) {
      if (newValue > 2015) {
        this._year = newValue;
        this.edition += newValue - 2015;
      }
    }
  }
});

var descriptor = Object.getOwnPropertyDescriptor(book, "_year");
alert(descriptor.value); // 2016
alert(descriptor.configurable); // false;
alert(typeof descriptor.get); // "undefined"

var descriptor = Object.getOwnPropertyDescriptor(book, "year");
alert(descriptor.value); // undefined
alert(descriptor.configurable); // false;
alert(typeof descriptor.get); // "function"
```

8. 创建对象

  1) 工厂模式

  ```javascript
  function createPerson(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function() {
      alert(this.name);
    };
    return o;
  }

  var person1 = createPerson("Nicholas", 29, "Software Engineer");
  ```

  工厂的模式缺点在于其解决了多个相似对象问题，但是并没有解决对象识别的问题（即知晓对象的类型）

  2）构造函数模式

  ```javascript
  function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function() {
      alert(this.name);
    };
  }

  var person1 = new Person("Nicholas", 29, "Software Engineer");
  ```

  这种模式并没有显式地创建对象，而且还直接将属性和方法赋予给了this对象。使用这种方法创建对象的话，必须使用`new`操作符，且实际上会经历4个步骤：创建一个新对象 -> 将构造函数的作用域赋给新对象 -> 执行构造函数的代码 -> 返回新对象。

  构造函数创建的对象有一个`constructor`属性指向Person。

  ```javascript
  alert(person1.constructor == Person); // true
  alert(person1 instanceof Object); // true
  alert(person1 instanceof Person); // true
  ```

  ---

  任何函数，只要通过new操作符来调用的话，它就可以作为构造函数。不然就是跟普通函数没什么区分的。Person()函数可以通过下面的任何一种方式来调用

  ```javascript
  // 当做构造函数
  var person = new Person("Nicholas", 29, "Software Engineer");
  person.sayName(); // Nicholas;

  // 普通函数
  Person("Nicholas", 29, "Software Engineer"); // 添加到了window
  window.sayName(); // Nicholas;

  // 在另一个对象的作用域中调用
  var o = new Object();
  Person.call(o, "Kristen", 23, "Nurse");
  o.sayName(); // Kristen
  ```
  ---

  构造函数模式的缺点在于每个方法都要在每个实例上重新创建一遍。

  3）原型模式

  每个函数都有一个`prototype`属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。

  ```javascript
  function Person() {}
  Person.prototype.name = "Nicholas";
  Person.prototype.age = 29;
  Person.prototype.job = "Software Engineer";
  Person.prototype.sayName = function() {
    alert(this.name);
  };

  var person1 = new Person();
  person1.sayName(); // Nicholas
  ```

  可以使用`hasOwnProperty()`来判断一个属性是来自实例还是来自原型，因为实例属性会屏蔽掉原型属性。如果同时使用`hasOwnProperty()`和`in`操作符的话，可以确定某个属性是存在于对象，还是原型中：

  ```javascript
  function hasPrototypeProperty(object, name) {
    return !object.hasPrototypeProperty(name) && name in object;
  }
  ```

  要取得对象上所有可枚举的实例属性，可以使用`Object.keys()`方法，该方法接收一个对象作为参数，返回一个包含所有可枚举属性的字符串数组。而使用`Object.getOwnPropertyNames()`则获取所有实例属性。

  ```javascript
  var keys = Object.getOwnPropertyNames(Person.prototype);
  alert(keys); // constructor, name, age, job, sayName
  ```

  原型模式的缺点在于所有实例都将取得相同的值，所有属性都被很多实例进行共享。

  4）组合使用构造函数模式和原型模式

  构造函数模式用于定义实例属性，原型模式用于定义方法和共享属性

  ```javascript
  function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.friends = ["Shelby", "Court"];
  }

  Person.prototype = {
    constructor: Person,
    sayName: function() {
      alert(this.name);
    }
  }
  ```

  5) 动态原型模式

  通过检查某个应该存在的方法是否有效，来决定是否需要初始化原型

  ```javascript
  function Person(name, age, job) {
    // 属性
    this.name = name;
    this.age = age;
    this.job = job;

    // 方法
    if (typeof this.sayName != "function") {
      Person.prototype.sayName = function() {
        alert(this.name);
      };
    }
  }
  ```

  需要注意的是，如果在已经创建实例的情况下重写原型，那么会切断所有现有的实例和原型之间的联系。

  6）寄生原型模式

  仅仅是封装创建对象的代码，然后再返回新创建的对象。

  ```javascript
  function Person(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function() {
      alert(this.name);
    };
    return o;
  }
  ```

9. 给原型添加方法的代码一定要放在替换原型的语句之后。另外，如果是利用原型链实现继承时，不能使用对象字面量创建原型方法。因为这样做的话，就会重写原型链。

10. 寄生组合继承

```javascript
function inheritPrototype(subType, superType) {
  var prototype = object(superType.prototype); // 创建对象
  prototype.constructor = subType;  // 增强对象
  subType.prototype = prototype; // 指定对象
}
```

11. 块级作用域(私有作用域)

```javascript
(function() {

})();
```

12. 全局变量不能通过`delete`删除，而直接在window对象上定义的属性可以

```javascript
var age = 29;
window.color = "blue";

delete window.age; // return false
delete window.color; // return true
```

13. 获取窗口左边和上边的位置

```javascript
var leftPos = (typeof window.screenLeft == "number") ? window.screenLeft : window.screenX;
var topPos = (typeof window.screenTop == "number") ? window.screenTop : window.screenY;
```

14. location对象的属性

|属性|例子|说明|
|:---:|:---:|:---:|
|hash|"#contents"|返回URL中的hash（#号后跟0或多个字符）|
|host|"www.example.com:80"|返回服务器名和端口号|
|hostname|"www.example.com"|返回不带端口号的服务器名称|
|href|"http://www.example.com"|返回当前加载页面的完整URL。而location的toString()也返回这个值|
|pathname|"/Wiley/"|返回URL中的目录和方法名|
|port|"8080"|返回URL中指定的端口号|
|protocol|"http:"|返回页面使用协议|
|search|"?q=javascript"|返回URL中的查询字符串|

15. 解析查询字符串

```javascript
function getQueryStringArgs() {
  // 取得查询字符串并去掉开头的问号
  var qs = (location.search.length > 0 ? location.search.substring(1) : "");
  // 保存数据的对象
  args = {};

  // 取得每一项
  items = qs.length ? qs.split("&") : [], item = null, name = null, value = null, i = 0, len = items.length
  for (i = 0; i < len; i++) {
    item = items[i].split("=");
    name = decodeURIComponent(item[0])
    value = decodeURIComponent(item[1])
    if (name.length) {
      args[name] = value;
    }
  }
  return args;
}
```

16. `cloneNode()`表示是否对节点进行复制。传入参数为true时为深复制，即复制节点及其整个子文档树。反之则为浅复制，只复制节点本身。

17. 取得HTML元素

比方说:

```HTML
<div id="myDiv" class="bd" title="Body text" lang="en" dir="lr"></div>
```

```javascript
var div = document.getElementById("myDiv");
alert(div.id); // myDiv
alert(div.className); // bd
alert(div.title); // Body text
alert(div.lang); // en
alert(div.dir); // dir
```

同时也可以为这些属性赋予新值。

18. 修改特性：`getAttribute()`, `setAttribute()`和`removeAttribute()`

```javascript
var div = document.getElementById("myDiv");
alert(div.getAttribute("id")); // myDiv
alert(div.getAttribute("class")); // bd
alert(div.getAttribute("title")); // Body text
alert(div.getAttribute("lang")); // en
alert(div.getAttribute("dir")); // dir
```

19. 在HTML5中，自定义的属性需要加上data-前缀

20. `arguments`转数组`Array.prototype.slice.call(arguments);`

21. 对于文本节点来说，可以通过nodeValue或者data属性来访问Text节点中包含的文本。（空格也可以当做是一个文本节点）。使用`createTextNode()`创建文本节点，`nomalize()`用于合并某个父元素中的多个文本节点，`splitText()`用于分割

22. Comment类型表示的是注释，其nodeValue是注释的内容。（Comment类型很少使用）

23. `querySelector()`接收一个CSS选择符，返回与该模式匹配的第一个元素。`querySelectorAll()`返回的是一个NodeList实例

24. 元素间的空格，通常浏览器都会返回文本节点

25. 元素遍历可以使用：

  1) `childElementCount` -- 返回子元素（不包括文本节点和注释）
  2） `firstElementChild` -- 指向第一个元素
  3) `lastElementChild` -- 指向最后一个元素
  4） `previousElementSibling` -- 指向前一个同辈元素
  5) `nextElementSibling` -- 指向后一个同辈元素

26. HTML5为所有元素添加了classList属性，方便操作类名：

  1）`add(value)` -- 将给定的字符串添加到列表，如果值已经存在，就不添加.
  2）`contains(value)` -- 表示列表中是否存在给定的值
  3）`remove(value)` -- 从列表中删除给定的字符串
  4）`toggle(value)` -- 如果列表中已经存在给定的值，则删除；如果没有，就添加

27. DOM中获得焦点的元素: `document.activeElement`

28. HTML5允许添加非标准的属性，只要是以“data-”开头的即可

  ```html
  <div id="myDiv" data-appId="12345" data-myName="Nicholas"></div>
  ```

  ```javascript
  // 取得appId
  var appId = div.dataset.appId;
  // 取得Name
  var myName = div.dataset.myName;
  ```

29. 删除事件处理程序的话，只需要将事件赋值为null即可。

30. DOM2级中的`addEventListener()`和`removeEventListener()`接收三个参数：要处理的事件名，作为事件处理的函数和一个布尔值。布尔值为true，表示在捕获阶段调用事件处理程序。为false的话表示在冒泡阶段调用。

31. 跨浏览器的事件处理程序

```javascript
// 先判断DOM2级方法，再判断IE方法，最后是DOM0级方法
var EventUtil = {
  addHandler: function(element, type, handler) {
    if (element.addEventListener) element.addEventListener(type, handler, false);
    else if (element.attachEvent) {
      element.attachEvent("on"+type, handler);
    } else element["on"+type] = handler;
  },
  removeHandler: function(element, type, handler) {
    if (element.removeEventListener) element.removeEventListener(type, handler, false);
    else if (element.detachEvent) {
      element.detachEvent("on"+type, handler);
    } else element["on"+type] = null;
  }
}
```

32. 阻止特定事件的默认行为，可以使用`preventDefault()`方法。也可以设置returnValue的值

```javascript
var link = document.getElementById("myLink");
link.onclick = function() {
  window.event.returnValue = false;
};
```
