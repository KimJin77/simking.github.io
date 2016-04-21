---
layout:     post
title:      "在Swift中使用C（译）"
date:       2015-04-08
author:     "Sim"
catalog: true
tags:
    - C
    - Swift
    - Translation
---


今天在做FFMPEG框架的时候遇到很多关于C的接口，一开始不知道如何使用，最后在网上找到这篇文章，写的比较详细，就试着翻翻了。

-----

Swift的类型系统通过严格的规则来使我们能够写出更少量的代码来完成更复杂的功能。但是在使用基于C的类库时，你就会发现那是各种不方便的。事实上，很多C的类库在Swift编译器上用起来，都是比较麻烦的。虽然Swift团队已经为使用C做出了很多努力，但是在Swift上使用C语言仍然是有很多的问题的。下面我们将探讨如何解决这些问题。

在我们开始之前，需要提醒大家的是，本文中大多数操作在绕过Swift编译的类型系统时，本质上都是不安全的。所以我建议大家仔细阅读并且不要复制粘贴本文的代码。这里不是Stack Overflow，这些代码段可能会损坏内存，引起内存泄漏，最坏的是引起程序崩溃。

## 基础

C指针在Swift中有两种表达方式

```Swift
UnsafePointer<T>
```

或者

```Swift
UnsafeMutablePointer<T>
```


`T`指的是Swift的类型中的原始C的类型。C语言中, 声明为`const`的指针就是`UnsafePointer`，否则就是`UnsafeMutablePointer`


下面举个栗子

```Swift
C:
void myFunction(const int *myConstIntPointer);

Swift:
func myFunction(myConstIntPointer: UnsafePointer<Int32>)

C:
void myOtherFunction(unsigned int *myUnsignedIntPointer);

Swift:
func myOtherFunciton(myUnsignedIntPointer: UnsafeMutablePointer<UInt32>)

C:
void iTakeAvoidPointer(void *aVoidPointer);

Swift:
func iTakeAvoidPointer(aVoidPointer: UnsafeMutablePointer<Void>)
```

如果Swift不清楚指针的类型的话，比方说一个提前声明，就可以使用`COpaquePointer`


```
C:
struct SomeThing
void iTakeAnOpaquePointer(struct SomeThing *someThing);

Swift:
func iTakeAnOpaquePointer(someThing: COpaquePointer)
```

## 将指针传递给Swift对象

在大多数例子中，基本上都会简单实用`inout`操作符，这就跟C中的`address-of`相似

```Swift
let myInt = 42
myFunction(&myInt)

var myUnsignedInt: UInt = 7
myOtherFunction(&myUnsignedInt)
```

有两个比较重要且微妙的细节：

1. 在使用`inout`的时候，使用`var`声明的变量和常量都会转换为`UnsafePointer`和`UnsafeMutablePointer`.如果不留意原先的类型的话，这很容易就会被遗忘，但当你将`UnsafePointer`传递到`UnsafeMutablePointer`的参数类型中却会出现error
2. 该操作符只会在上下文中传递Swift值和引用函数参数时生效。你不能在其他上下文中获取指针。比方说，下面的代码将会引起编译器错误：

```Swift
let x = 42
let y = &x
```

你需要时不时地和使用或者返回空指针来替代其他类型的API进行协作。这在指定通用类型的C语言中并不常见。

```C
void takesAnObject(void *theObject);
```

如果你知道函数所需要的参数的类型，你可以强迫某个对象使用`withUnsafePointer`和`unsafeBitCast`来变成指针。比方说，`takesAnObject`需要一个指向int类型的指针

```
var test = 42
withUnsafePointer(&test, { (ptr: UnsafePointer<Int>) -> Void in
	var voidPtr: UnsafePointer<Void> = unsafeBitCast(ptr, UnsafePointer<Void>.self)
	takesAnObject(voidPtr)
})
```

来分析下。首先，我们调用了`withUnsafeMutablePointer`。这个函数带着两个参数。第一个参数是`inout`的`T`，第二个是`(UnsafePointer) -> ResultType`类型的闭包。这个闭包将函数的第一个参数的指针作为唯一的参数。函数返回的是闭包的处理结果，对于上面的例子来说，返回了Void，也就是啥都没返回。所以，我们可以

```Swift
let ret = withUnsafePointer(&test, { (ptr: UnsafePointer<Int>) -> Int32 in
    var voidPtr: UnsafePointer<Void> = unsafeBitCast(ptr, UnsafePointer<Void>.self)
    return takesAnObjectAndReturnsAnInt(voidPtr)
})
print(ret)
```

**注意:**如果需要修改指针本身，可以使用`withUnsafeMutablePointer`

Swift也有方法是可以传递两个参数的

```
var x: Int = 7
var y: Double = 4
withUnsafePointers(&x, &y, { ptr1: UnsafePointer<Int>, ptr2: UnsafePointer<Double>) -> Void in
	var voidPtr1: UnsafePointer<Void> = unsafeBitCast(ptr1, UnsafePointer<Void>.self)
	var voidPtr2: UnsafePointer<Void> = unsafeBitCast(ptr2, UnsafePointer<Void>.self)
	takesTwoPointers(voidPtr1, voidPtr2)
})
```

### 关于unsafeBitCast

`unsafeBitCast`是个极度危险的操作。文档的描述中提到“残忍将某样东西强行转化为同样大小的另外一样物体”。我们之所以能够安全地使用它，是因为我们只是简单的进行指针类型的转换，并且所有指针的大小都是一样的.这就是为什么我们在转换之前需要调用`withUnsafePointer`来获取`UnsafePointer`的类型。

很容易犯的错误是

```
var x: Int = 7
let xPtr = unsafeBitCast(x, UnsafePointer<Void>.self)
```

上面的代码段想要获取指向x的指针。但是这将会起到误导的作用。因为它将会编译并且运行，最后获取到的结果是0x7或者垃圾指针，而不是我们想要的指向x的指针。

因为`unsafeBitCast`转换时size必须相等。

```
var x: Int8 = 7
let xPtr = unsafeBitCast(x, UnsafePointer<Void>.self)
```

## 和C的结构体进行交互

假设你想要取得你的电脑正在运行着的系统的信息。使用C的话，有个API`uname(2)`，可以获取关于你系统的信息，比方说OS名称和版本等。用Swift表达起来就是

```C
struct utsname {
	var sysname: (Int8, Int8, ...253 times..., Int8)
	var nodename: (Int8, Int8, ...253 times..., Int8)
	var release: (Int8, Int8, ...253 times..., Int8)
	var version: (Int8, Int8, ...253 times..., Int8)
	var machine: (Int8, Int8, ...253 times..., Int8)
}
```

用Swift实现起来的话，就是

```Swift
var name = utsname(sysname:(0, 0, 0, ..., 0), nodename: :(0, 0, 0, ..., 0), etc)
utsname(&name)

var machine = name.machine
print(machine)
```

这同时也有另一个问题，就是`machine`是个元组，所以打印出来的是256个Int8类型的，但是只有少数一部分ASCII的值使我们想要获取的。

### 所以，怎么办？

Swift的`UnsafeMutablePointer`支持两个方法，`alloc(Int)`和`dealloc(Int)`。

```Swift
let name = UnsafeMutablePointer<utsname>.alloc(1)
uname(name)

let machine = withUnsafePointer(&name.memory.machine, { (ptr) -> String? in
	let int8Ptr = unsafeBitCast(ptr, UnsafePointer<Int8>.self)
	return String.fromCString(int8Ptr)
})

name.dealloc(1)
if let m = machine {
	print(m)
}

```

调用`withUnsafePointer`，然后将元组作为参数进行传递并且由闭包返回一个可选的String

在闭包中，我们使用UnsafePointer转换指针。因为Swift的String有将`CChar`强制转换的类方法，所以可以将新的指针传递给初始化操作并且获取返回值。

## 总结

使用unsafe API的话，还是需要记住这是个不安全的操作。

（后面的就没啥好翻出来的了）


原文链接：[Using Legacy C APIs with Swift](http://www.sitepoint.com/using-legacy-c-apis-swift/)
