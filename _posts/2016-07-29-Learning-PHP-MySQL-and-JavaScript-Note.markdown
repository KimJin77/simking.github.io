---
layout:     post
title:      "Learning PHP, MySQL & JavaScript学习笔记"
date:       2016-07-29
author:     "Sim"
catalog: true
tags:
    - PHP
    - MySQL
    - JavaScript
---

# PHP
1. PHP已定义的常量

| 常量 | 描述 |
| --- | --- |
| __LINE__ | 行号 |
| __FILE__ | 当前文件的完整路径 |
| __DIR__ | 文件的目录 |
| __FUNCTION__ | 函数名称 |
| __CLASS__ | 类名 |
| __METHOD__ | 类的方法名 |
| __NAMESPACE__ | 当前的命名空间  |

2. `echo`是一个纯粹的PHP语言结构输出，而`print`则像是一个带着单个参数和返回值的函数（返回值一般为1）
3. 超级全局变量

| 变量名 | 内容 |
| --- | --- |
| `$GLOBALS` | 脚本中全局范围内定义的变量 |
| `$_SERVER` | 包含了头文件，路径和脚本位置等信息 |
| `$_GET` | 脚本通过HTTP GET方法传送过来的变量 |
| `$_POST` | 脚本通过HTTP POST方法传送过来的变量 |
| `$_FILES` | 脚本通过HTTP POST方法传送过来的文件 |
| `$_COOKIE` | 脚本通过HTTP cookies传递的变量 |
| `$_SESSION` | 当前脚本的Session变量 |
| `$_REQUEST` | 由浏览器传递过来的内容，默认情况下，指的是`$_GET`,`$_POST`和`$_COOKIE` |
| `$_ENV` | 通过环境方法传递给脚本的变量 |

需要注意的是，由于超级全局变量的使用容易使网站容易被攻击，所以通常会在使用超级全局变量之前对其使用`htmlentities`方法进行处理。该方法将所有的字符转换为HTML的实体。举个例子，

```PHP
$came_from = htmlentities($_SERVER['HTTP_REFERER']);
```

4. PHP中false为NULL，输出时不会输出为0

```PHP
<?php
	echo 'a: ['.TRUE.']<br>';
	echo 'b: ['.FALSE.']<br>';
?>

// result
a: [1]
b: []
```

5. PHP的Object是引用类型的。所以如果要复制一个对象的话，使用`clone`。

```php
<?php
	$object1 = new User();
	$object1->name = 'Alice`;
	$object2 = clone $object1;
?>
```

6. 构造器

```php
<?php
	class User {
		function User($param1, $param2) {
			public $username = 'Guest`;
		}
	}
?>
```

PHP 5之后的构造器

```php
<?php
	class User {
		function __construct($param1, $param2) {
			public $username = 'Guest`;
		}
	}
?>

```

7. 在类中声明属性时，如果需要分配初始值的话，该值只能是常量，而不能是表达式或者是函数。
8. 调用父类的方法，使用双冒号

```php
function test() {
	parent::test2();
}
```

9. 子类构造器

```php
class Children extends Father {
	function __construct() {
		parent::__construct();
		// init property of Children
	}
}
```

10. 如果不想某个方法被子类继承的话，使用final修饰
11. 遍历数组

```php
$paper = array('copier' => "Copier & Multipurpose",
					'inkjet' => "Inkjet Printer",
					'laser' => "Laser Printer",
					'photo' => "Photographic Paper");
					
foreach($paper as $item => $description) {
	...
}

// Or
while(list($item, $description) = each($paper)) {
	...
}
```

12. 时间格式

e.g

```php
echo date("l F jS, Y - g:ia", time());

// Thursday July 6th, 2017 - 1: 38pm
```

| 格式 | 描述 | 返回值 |
| --- | --- | --- |
| d | 月份天数，两位数 | 01 - 30 |
| D | 星期几 | Mon - Sun |
| j | 月份天数，不以0开头 | 1 - 30 |
| l | 星期几，全称 | Monday - Sunday |
| N | 星期几，数字形式 | 1 - 7 |
| S | 数字后缀，常与j一起使用 | st, nd, rd, th |
| w | 星期几，周日到周六，数字形式 | 0 - 6 |
| z | 年份天数 | 0 - 365 |
| W | 年份星期数 | 0 - 52 |
| F | 月份名称 | January - December |
| m | 月份 | 01 - 12 |
| M | 月份名称简写 | Jan - Dec |
| n | 月份，不以0开头 | 1 - 12 |
| t | 给定月份的天数 | 28 - 31 |
| L | 是否闰年 | 1 = Yes, 0 = No |
| y | 年份，2位数 | 00 - 99 |
| Y | 年份，4位数 | 0000 - 9999 |
| a | 上午/下午， 小写 | am/pm |
| A | 上午/下午， 大写 | AM/PM |
| g | 12小时制 | 1 - 12 |
| G | 24小时制 | 00 - 23 |
| h | 12小时制 | 01 - 12 |
| H | 24小时制 | 00 - 23 |
| i | 分钟 | 00 - 59 |
| s | 秒钟 | 00 - 59 |

13. 


