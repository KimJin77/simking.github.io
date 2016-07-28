---
layout:     post
title:      "Airbnb的JavaScript风格指南（译）"
date:       2016-07-21
author:     "Sim"
catalog: true
tags:
    - JavaScript
    - 译文
---

# 类型
* 原始数据： 直接存取其自身
	* `string`
	* `number`
	* `boolean`
	* `null`
	* `undefined`

	```JavaScript
	const foo = 1;
	let bar = foo;
	
	bar = 9;
	
	console.log(foo, bar); // => 1, 9
	```
	
* 复杂类型：存取时使用的是其值的引用
	* `object`
	* `array`
	* `function`
	
	```JavaScript
	const foo = [1, 2];
	const bar = foo;
	
	bar[0] = 3;
	
	console.log(foo[0], bar[0]); // => 3, 3
	```
	
# 对象
* 使用字面量来创建对象

```JavaScript
// bad
const item = new Object();

// good
const item = {};
```

* 不要使用保留字作为键名，因为在IE8下将无法生效

```JavaScript
// bad
const superman = {
	default: { clark: 'kent' },
	private: true,
};

// good
const superman = {
	defaults: { clark: 'kent' },
	hidden: true,
};
```

* 使用同义词来替代保留字

```JavaScript
// bad
const superman = {
	class: 'alien',
};

// bad 
const superman = {
	klass: 'alien',
};

// good 
const superman = {
	type: 'alien',
};
```

* 创建拥有动态属性的对象时使用计算属性

```JavaScript
function getKey(k) {
	return 'a key named ${k}';
};

// bad
const obj = {
	id: 5,
	name: 'San Francisco',
};
obj[getKey('enabled')] = true;

// good
const obj = {
	id: 5,
	name: 'San Francisco',
	[getKey('enabled')]: true
};
``` 

* 简写对象方法

```JavaScript
// bad
const atom = {
	value: 1,
	addValue: function(value) {
		return atom.value + value;
	},
};

// good
const atom = {
	value: 1,
	addValue(value) {
		return atom.value + value;
	},
};
```

* 简写属性名称

```JavaScript
const lukeSkywalker = 'Luke Skywalker';

// bad
const obj = {
	lukeSkywalker: lukeSkywalker,
};

// good
const obj = {
	lukeSkywalker,
};
```

* 将简写的属性放到对象声明的起始处

```JavaScript
const anakinSkywalker = 'Anakin Skywalker';
const lukeSkywalker = 'Luke Skywalker';

// bad 
const obj = {
	episode: 1,
	twoJediWalkIntoACantina: 2,
	lukeSkywalker,
	episodeThree: 3,
	mayTheFourth: 4,
	anakinSkywalker,
};

// good
const obj = {
	lukeSkywalker,
	anakinSkywalker,
	episode: 1,
	twoJediWalkIntoACantina: 2,
	episodeThree: 3,
	mayTheFourth: 4,
};
```

* 在属性名使用了无效的标识符时，给属性名使用引号

```JavaScript
// bad
const bad = {
	'foo': 3,
	'bar': 4,
	'data-blah': 5,
};

// good
const good = {
	foo: 3,
	bar: 4,
	'data-blah': 5,
};
```

* 不要直接调用`Object.prototype`的方法，比如`hasOwnProperty`, `propertyIsEnumerable`和`isPropertypeOf`

```JavaScript
// bad
console.log(object.hasOwnProperty(key));

// good
console.log(Object.prototype.hasOwnProperty.call(object, key));

// best
const has = Object.prototype.hasOwnProperty; // 先获取缓存对象
/* or */
const has = require('has');
...
console.log(has.call(object, key));
```

# 数组
* 使用字面量进行初始化

```JavaScript
// bad
const items = new Array();

// good 
const items = [];
```

* 使用`Array#push`来添加对象

```JavaScript
const someStack = [];

// bad
someStack[someStack.length] = 'abcdefg';

// good
someStack.push('abde');
```

* 使用`...`来复制数组

```JavaScript
// bad 
const len = items.length
const itemsCopy = [];
let i;

for (i = 0; i < len; i++) {
	itemsCopy[i] = items[i];
}

// good
const itemsCopy = [...items];
```

* 将类数组对象转换成数组的话使用`Array.from`

```JavaScript
const foo = document.querySelectorAll('.foo');
const nodes = Array.from(foo);
```

* 在数组方法的回调中使用return

```JavaScript
// good
[1, 2, 3].map((x) => {
	const y = x + 1;
	return x * y;
};

// good
[1, 2, 3].map(x => x + 1);

// bad
const flat = {};
[[0, 1], [2, 3], [4, 5]].reduce((memo, item, index) => {
	const flattern = memo.concat(item);
	flat[index] = flattern;
});

// good
const flat = {};
[[0, 1], [2, 3], [4, 5]].reduce((memo, item, index) => {
	const flattern = memo.concat(item);
	flat[index] = flattern;
	return flattern;
});

// bad
inbox.filter((msg) => {
	const { subject, author } = msg;
	if (subject === 'Mockingbird') {
		return author === 'Harper Lee';
	} else {
		return false;
	}
});

// good
inbox.filter((msg) => {
	const { subject, author } = msg;
	if (subject === 'Mockingbird') {
		return author === 'Harper Lee';
	} 
	return false;
});
```

# 重构
* 在访问和使用对象的多个属性时，使用对象重构

```JavaScript
// bad
function getFullName(user) {
	const firstName = user.firstName;
	const lastName = user.lastName;
	
	return '${firstName} ${lastName}';
}

// good
function getFullName(user) {
	const { firstName, lastName } = user;
	return '${firstName} ${lastName}';
}

// best
function getFullName({ firstName, lastName}) {
	return '${firstName} ${lastName}';
}
```

* 数组重构

```JavaScript
const arr = [1, 2, 3, 4];

// bad
const first = arr[0];
const second = arr[1];

// good
const [first, second] = arr;
```

* 在有多个返回值的时候使用对象重构，而不是数组重构

```JavaScript
// bad
function processInput(input) {
	// then a miracle occurs
	return [left, right, top, bottom];
}	

// the caller needs to think about the order of return data
const [left, __, top] = processInput(input);

// good
function processInput(input) {
	// then a miracle occurs
	return { left, right, top, bottom };
}

// the caller selects only the data they need
const { left, top } = processInput(input);
```

# 字符串
* 使用单引号

```JavaScript
// bad 
const name = "Capt. Janeway";

// good
const name = 'Capt. Janeway';
```

* 超过100个字符的字符串使用连接符写成多行
* 如果过度使用长字符串的话，可能导致性能问题

```JavaScript
// bad
const errorMessage = 'This is a super long error that was thrown because of Batman. When you stop to think about how Batman had anything to do with this, you would get nowhere fast.';

// bad
const errorMessage = 'This is a super long error that was thrown because \
of Batman. When you stop to think about how Batman had anything to do \
with this, you would get nowhere \
fast.';

// good
const errorMessage = 'This is a super long error that was thrown because ' +
  'of Batman. When you stop to think about how Batman had anything to do ' +
  'with this, you would get nowhere fast.';
```

* 程序化生成的字符串，使用模板字符来替代连接符

```JavaScript
// bad
function sayHi(name) {
	return 'How are you, ' + name + '?';
}

// bad
function sayHi(name) {
	return ['How are you', name, '?'].join();
}

// bad
function sayHi(name) {
	return 'How are you, ${ name }?';
}

// good
function sayHi(name) {
	return 'How are you, ${name}?';
}
```

* 不要在字符串上使用`eval()`方法，它有着太多的漏洞
* 尽量不要使用转义字符

```JavaScript
// bad
const foo = '\'this\' \i\s \"quoted\"';

// good
const foo = '\'this\' is "quoted"';
const foo = 	`'this' is 'quoted'`;
```

# 函数
* 使用函数声明	来替代函数表达式

```JavaScript
// bad 
const foo = function() {
};

// good
function foo() {
};
```

* 立即调用的函数表达式

```JavaScript
(function () {
	console.log('Welcome to the Internet. Please follow me.');
}());
```

* 不要在非函数块中（if, while等）声明模块。作为替代，可以将函数分配给某个变量。
* ECMA-262将`block`定义为描述语句列表。并且，函数声明不属于描述语句

```JavaScript
// bad
if (currentUser) {
	function test() {
		console.log('Nope.');
	}
}

// good
let test;
if (currentUser) {
	test = () => {
		console.log('Yup.');
	};
}
```

* 不要将变量声明为`arguments`，它将会替代函数范围中的`arguments`对象

```JavaScript
// bad
function nope(name, options, arguments) {
	// ...stuff...
}

// good
function yup(name, options, args) {
	// ...stuff...
}
```

* 使用`...`来替代`arguments`

```JavaScript
// bad
function concatenateAll() {
	const args = Array.prototype.slice.call(arguments);
	return args.join('');
}

// good
function 	concatenateAll(...args) {
	return args.join('');
}
```

* 使用默认的参数语法来替代可变的函数参数

```JavaScript
// really bad
function handleThings(opts) {
	opts = opts || {};
}

// still bad
function handleThings(opts) {
	if (opts === void 0) {
		opts = {};
	}
	// ...
}

// good 
function handleThings(opts = {}) {
	// ...
}
```

* 避免默认参数值的影响

```JavaScript
var b = 1;
// bad
function count(a = b++) {
	console.log(a);
}

count(); // 1
count(); // 2
count(3); // 3
count(); // 3 
```

* 将带有缺省值的参数放到最后

```JavaScript
// bad 
function handleThings(opts = {}, name) {
	// ...
}

// good
function handleThings(name, opts = {}) {
	// ...
}
```

* 不要使用函数构造器来创建新的函数

```JavaScript
// bad
var add = new Function('a', 'b', 'return a + b');

// still bad
var subtract = Function('a', 'b', 'return a - b');
```

* 函数签名的空格

```JavaScript
// bad
const f = function() {};
const g = function (){};
const h = function(){};

// good
const x = function () {};
const y = function a() {};
```

* 不要改变参数

```JavaScript
// bad
function f1(obj) {
	obj.key = 1;
}

// good
function f2(obj) {
	const key = Object.prototype.hasOwnProperty.call(obj, 'key') ? obj.key : 1;
}
```

* 不重新分配参数

```JavaScript
// bad
function f1(a) {
	a = 1;
}

function f2(a) {
	if (!a) { a = 1; }
}

// good
function f3(a) {
	const b = a || 1;
}

function f4(a = 1) {
}
```

# 数组函数
* 使用函数表达式时使用`=>`

```JavaScript
// bad 
[1, 2, 3].map(function (x) {
	const y = x + 1;
	return x * y;
});

// good
[1, 2, 3].map((x) => {
	const y = x + 1;
	return x * y;
});
```

* 如果函数由单个表达式构成的话，可以省略大括号并使用隐式的`return`表达式

```JavaScript
// bad
[1, 2, 3].map(number => {
	const nextNumber = number + 1;
	'A string 	containing the ${nextNumber}.';
});

// good
[1, 2, 3].map(number => 'A string containing the ${number}.');

// good 
[1, 2, 3].map(number => {
	const nextNumber = number + 1;
	return 'A string 	containing the ${nextNumber}.';
});

// good
[1, 2, 3].map((number, index) => ({
	index: number
}));
```

* 用括号将过长的表达式括起来增加可阅读性

```JavaScript
// bad
[1, 2, 3].map(number => 'As time went by, the string containing the ' +
  `${number} became much longer. So we needed to break it over multiple ` +
  'lines.'
);

// good
[1, 2, 3].map(number => (
  `As time went by, the string containing the ${number} became much ` +
  'longer. So we needed to break it over multiple lines.'
));
```

* 如果函数只带了一个参数并且没大括号括起来的话，就省略掉括号

```JavaScript
// bad
[1, 2, 3].map((x) => x * x);

// good
[1, 2, 3].map(x => x * x);

// good
[1, 2, 3].map(number => (
  `A long string with the ${number}. It’s so long that we’ve broken it ` +
  'over multiple lines!'
));

// bad
[1, 2, 3].map(x => {
  const y = x + 1;
  return x * y;
});

// good
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});
```

# 类和构造器
* 尽量使用`class`，避免直接使用`prototype`

```JavaScript
// bad 
function Queue(contents = []) {
	this.queue = [...contents];
}

Queue.prototype.pop = function () {
	const value = this.queue[0];
	this.queue.splice(0, 1);
	return value;
}

// good
class Queue {
	constructor(contents = []) {
		this.queue = [...contents];
	}
	pop() {
		const value = this.queue[0];
		this.queue.slice(0, 1);
		return value;
	}
}
```

* 继承时使用`extends`

```JavaScript
// bad
const inherits = require('inherits');
function PeekableQueue(contents) {
	Queue.apply(this, contents);
}

inherits(PeekableQueue, Queue);
PeekableQueue.prototype.peek = function () {
	return this._queue[0];
}

// good
class PeekableQueue extends Queue {
	peek() {
		return this._queue[0];
	}
}
```

* 方法体返回`this`有利于方法链接

```JavaScript
// bad
Jedi.prototype.jump = function () {
	this.jumping = true;
	return true;
}

Jedi.prototype.setHeight = function (height) {
	this.height = height;
}

// good
class Jedi {
	jump() {
		this.jumping = true;
		return this;
	}
	
	setHeight() {
		this.height = height;
		return this;
	}	
}

const luke = new Jedi();

luke.jump()
	.setHeight(20);
```

# 模块
* 在非标准化模块系统中使用`import`/`export`

```JavaScript
// bad
const AirbnbStyleGuide = require('./AirbnbStyleGuide');
module.exports = AirbnbStyleGuide;

// good
import AirbnbStyleGuide from './AirbnbStyleGuide';
export default AirbnbStyleGuide.es6;

// best
import { es6 } from './AirbnbStyleGuide';
export default es6;
```

* 不要使用通用符号

```JavaScript
// bad 
import * as AirbnbStyleGuide from './AirbnbStyleGuide';

// good
import AirbnbStyleGuide from './AirbnbStyleGuide'; 
```

* 不要直接使用`import`导入文件

```JavaScript
// bad
// filename es6.js
export { es6 as default } from './AirbnbStyleGuide';

// good
import { es6 } from './AirbnbStyleGuide';
export default es6;
```

* 同个路径中的文件导入一次即可

# 遍历和生成
* 不要使用遍历。选择JS中提供的方法来替代`for-in`等遍历方法

```JavaScript
const numbers = [1, 2, 3, 4, 5];

// bad
let sum = 0;
for (let num of numbers) {
	sum += num;
}

sum === 15;

// good
let sum = 0;
number.forEach(num => sum += num);

// best
const sum = number.reduce((total, num) => total + num, 0);
```

# 属性
* 使用点语法

```JavaScript
const luke = {
	jedi: true,
	age: 28,
};

// bad
const isJedi = luke['jedi'];

// good
const isJedi = luke.jedi;
```

* 通过变量访问属性时使用`[]`

```JavaScript
const luke = {
	jedi: true,
	age: 28,
};

function getProp(prop) {
	return luke[prop];
}

var isJedi = getProp('jedi');
```

# 变量
* 除了全局变量以外，一般使用`const`来声明变量

```JavaScript
// bad
superpower = new SuperPower();

// good
const superpower = new Superpower();
```

* 在需要用到变量时才进行声明

# 提升
* `var`的声明会提升到作用域顶部，但是赋值却不会

```JavaScript
// we know this wouldn't work (assuming there
// is no notDefined global variable)
function example() {
  console.log(notDefined); // => throws a ReferenceError
}

// creating a variable declaration after you
// reference the variable will work due to
// variable hoisting. Note: the assignment
// value of `true` is not hoisted.
function example() {
  console.log(declaredButNotAssigned); // => undefined
  var declaredButNotAssigned = true;
}

// the interpreter is hoisting the variable
// declaration to the top of the scope,
// which means our example could be rewritten as:
function example() {
  let declaredButNotAssigned;
  console.log(declaredButNotAssigned); // => undefined
  declaredButNotAssigned = true;
}

// using const and let
function example() {
  console.log(declaredButNotAssigned); // => throws a ReferenceError
  console.log(typeof declaredButNotAssigned); // => throws a ReferenceError
  const declaredButNotAssigned = true;
}
```

* 匿名函数表达式会提升变量名，但是不会提升函数赋值

```JavaScript
function example() {
	console.log(anonymous); // => undefined
	
	anonymous(); // => TypeError anonymous is not a function
	
	var anonymous = function() {
		console.log('anonymous function expression');
	};
}
```

* 函数表达式提升了变量名，但不会提升函数名和函数主体

```JavaScript
function example() {
  console.log(named); // => undefined

  named(); // => TypeError named is not a function

  superPower(); // => ReferenceError superPower is not defined

  var named = function superPower() {
    console.log('Flying');
  };
}

// the same is true when the function name
// is the same as the variable name.
function example() {
  console.log(named); // => undefined

  named(); // => TypeError named is not a function

  var named = function named() {
    console.log('named');
  }
}
```

* 函数声明提升了它们的名称和函数体

```JavaScript
function example() {
	superPower(); // => Flying
	
	function superPower() {
		console.log('Flying');
	}
}
```

# 比较运算符&等号
* 优先使用`===`和`!==`
* `if`等条件表达式通过抽象方法`ToBoolean`强制计算表达式时总是遵守下面的规则
	* 对象为true
	* Undefined为false
	* Null为false
	* 布尔值不变
	* 数字如果是+0, -0或者是NaN的话为false， 其他为true
	* 字符串如果是空字符串则为false
* 使用简写

```JavaScript
// bad
if (name !== '') { 
	// stuff...
}

// good
if (name) {
	// ...stuff...
}

// bad
if (collection.length > 0) {
	// ...stuff...
}

// good
if (collection.length) {
	// ...stuff...
}
```

* switch中的条件用`{}`包括起来

```JavaScript
 // bad
  switch (foo) {
    case 1:
      let x = 1;
      break;
    case 2:
      const y = 2;
      break;
    case 3:
      function f() {}
      break;
    default:
      class C {}
  }

  // good
  switch (foo) {
    case 1: {
      let x = 1;
      break;
    }
    case 2: {
      const y = 2;
      break;
    }
    case 3: {
      function f() {}
      break;
    }
    case 4:
      bar();
      break;
    default: {
      class C {}
    }
  }
```

* 三目运算符应该在同一行
* 去除不必要的运算符

# 块
* 使用大括号包裹多行代码

```JavaScript
// bad
if (test) 
	return false;

// good
if (test) return false;

// good
if (test) {
	return false;
}

// bad
function foo() { return false; }

// good 
function foo() {
	return false;
}
```

# 注释
* 多行注释的话使用`/**...*/`

```JavaScript
// bad
// make() returns a new element
// based on the passed in tag name
//
// @param {String} tag
// @return {Element} element
function make(tag) {

  // ...stuff...

  return element;
}

// good
/**
 * make() returns a new element
 * based on the passed in tag name
 *
 * @param {String} tag
 * @return {Element} element
 */
function make(tag) {
  // ...stuff...
  return element;
}
```

* 单行注释使用`//`。每个注释单独一行

```JavaScript
// bad
const active = true;  // is current tab

// good
// is current tab
const active = true;

// bad
function getType() {
  console.log('fetching type...');
  // set the default type to 'no type'
  const type = this._type || 'no type';

  return type;
}

// good
function getType() {
  console.log('fetching type...');

  // set the default type to 'no type'
  const type = this._type || 'no type';

  return type;
}

// also good
function getType() {
  // set the default type to 'no type'
  const type = this._type || 'no type';

  return type;
}
```

* 给注释增加 FIXME 或 TODO 的前缀可以帮助其他开发者快速了解这是一个需要复查的问题，或是给需要实现的功能提供一个解决方式。这将有别于常见的注释，因为它们是可操作的。使用 FIXME -- need to figure this out 或者 TODO -- need to implement。
* 使用`// FIXME:`标注问题

```JavaScript
function Calculator() {

  // FIXME: shouldn't use a global here
  total = 0;

  return this;
}
```

* 使用`// TODO:`标注问题的解决方式

```JavaScript
class Calculator extends Abacus {
  constructor() {
    super();

    // TODO: total should be configurable by an options param
    this.total = 0;
  }
}
```

# 空格
* 使用2个空格作为缩进
* 大括号前放一个空格
* 条件语句的小括号前放一个空格。方法调用和声明时方法名和小括号不需要放空格
* 操作符的前后都放空格
* 文件末尾插入空行
* 在使用长方法链时进行缩进。使用前面的点 . 强调这是方法调用而不是新语句。

```JavaScript
// bad
$('#items').find('.selected').highlight().end().find('.open').updateCount();

// bad
$('#items').
  find('.selected').
    highlight().
    end().
  find('.open').
    updateCount();

// good
$('#items')
  .find('.selected')
    .highlight()
    .end()
  .find('.open')
    .updateCount();

// bad
var leds = stage.selectAll('.led').data(data).enter().append('svg:svg').classed('led', true)
    .attr('width', (radius + margin) * 2).append('svg:g')
    .attr('transform', 'translate(' + (radius + margin) + ',' + (radius + margin) + ')')
    .call(tron.led);

// good
var leds = stage.selectAll('.led')
    .data(data)
  .enter().append('svg:svg')
    .classed('led', true)
    .attr('width', (radius + margin) * 2)
  .append('svg:g')
    .attr('transform', 'translate(' + (radius + margin) + ',' + (radius + margin) + ')')
    .call(tron.led);
```

* 在块末和新句之前插入空行
* 块声明和语句之间不要留空行

```JavaScript
// bad
function bar() {

  console.log(foo);

}

// also bad
if (baz) {

  console.log(qux);
} else {
  console.log(foo);

}

// good
function bar() {
  console.log(foo);
}

// good
if (baz) {
  console.log(qux);
} else {
  console.log(foo);
}
```

* 每行代码的长度不要超过100

# 逗号
* 不需要行首逗号

```JavaScript
// bad
const story = [
    once
  , upon
  , aTime
];

// good
const story = [
  once,
  upon,
  aTime,
];

// bad
const hero = {
    firstName: 'Ada'
  , lastName: 'Lovelace'
  , birthYear: 1815
  , superPower: 'computers'
};

// good
const hero = {
  firstName: 'Ada',
  lastName: 'Lovelace',
  birthYear: 1815,
  superPower: 'computers',
};
```

* 行末逗号

```JavaScript
// bad - git diff without trailing comma
const hero = {
     firstName: 'Florence',
-    lastName: 'Nightingale'
+    lastName: 'Nightingale',
+    inventorOf: ['coxcomb chart', 'modern nursing']
};

// good - git diff with trailing comma
const hero = {
     firstName: 'Florence',
     lastName: 'Nightingale',
+    inventorOf: ['coxcomb chart', 'modern nursing'],
};

// bad
const hero = {
  firstName: 'Dana',
  lastName: 'Scully'
};

const heroes = [
  'Batman',
  'Superman'
];

// good
const hero = {
  firstName: 'Dana',
  lastName: 'Scully',
};

const heroes = [
  'Batman',
  'Superman',
];
```

# 类型转换
* 在语句开始前执行转换
* 字符串

```JavaScript
// => this.reviewScore = 9;

// bad
const totalScore = this.reviewScore + ''; // invokes this.reviewScore.valueOf()

// bad
const totalScore = this.reviewScore.toString(); // isn't guaranteed to return a string

// good
const totalScore = String(this.reviewScore);
```

* Numbers

```JavaScript
const inputValue = '4';

// bad
const val = new Number(inputValue);

// bad
const val = +inputValue;

// bad
const val = inputValue >> 0;

// bad
const val = parseInt(inputValue);

// good
const val = Number(inputValue);

// good
const val = parseInt(inputValue, 10);
```

* 如果因为某些原因 parseInt 成为你所做的事的瓶颈而需要使用位操作解决性能问题时，留个注释说清楚原因和你的目的
* 布尔

```JavaScript
const age = 0;

// bad
const hasAge = new Boolean(age);

// good
const hasAge = Boolean(age);

// best
const hasAge = !!age;
```

# 命名规则
* 避免单个字母的命名
* 使用驼峰命名法来命名对象，函数和实例
* 使用帕斯卡式命名法命名构造器和类

```JavaScript
// bad
function user(options) {
  this.name = options.name;
}

const bad = new user({
  name: 'nope',
});

// good
class User {
  constructor(options) {
    this.name = options.name;
  }
}

const good = new User({
  name: 'yup',
});
```

* 不要在前后缀添加下划线
* 不要保存`this`的引用

# 存取器
* 存取器是非必需的
* 不要使用JS的getter/setter。使用`getVal()`和`setVal("hello")`来取代

```JavaScript
// bad
class Dragon {
  get age() {
    // ...
  }

  set age(value) {
    // ...
  }
}

// good
class Dragon {
  getAge() {
    // ...
  }

  setAge(value) {
    // ...
  }
}
```

* 如果属性/方法属于布尔类型的，使用`isVal()`或者是`hasVal()`

# 事件
* 当给事件附加数据时（无论是 DOM 事件还是私有事件），传入一个哈希而不是原始值。这样可以让后面的贡献者增加更多数据到事件数据而无需找出并更新事件的每一个处理器。例如，不好的写法：

```JavaScript
// bad
$(this).trigger('listingUpdated', listing.id);

...

$(this).on('listingUpdated', function (e, listingId) {
  // do something with listingId
});

// good
$(this).trigger('listingUpdated', { listingId : listing.id });

...

$(this).on('listingUpdated', function (e, data) {
  // do something with data.listingId
});
```

# jQuery
* jQuery对象的前缀为`$`
* 缓存jQuery对象

```JavaScript
// bad
function setSidebar() {
  $('.sidebar').hide();

  // ...stuff...

  $('.sidebar').css({
    'background-color': 'pink'
  });
}

// good
function setSidebar() {
  const $sidebar = $('.sidebar');
  $sidebar.hide();

  // ...stuff...

  $sidebar.css({
    'background-color': 'pink'
  });
}
```

* 对 DOM 查询使用层叠 $('.sidebar ul') 或 父元素 > 子元素 $('.sidebar > ul')
* 对有作用域的 jQuery 对象查询使用`find`

```JavaScript
// bad
$('ul', '.sidebar').hide();

// bad
$('.sidebar').find('ul').hide();

// good
$('.sidebar ul').hide();

// good
$('.sidebar > ul').hide();

// good
$sidebar.find('ul').hide();
```

