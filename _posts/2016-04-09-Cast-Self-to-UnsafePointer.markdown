---
layout:     post
title:      "在swift中将self转换成UnsafeMutablePointer<Void>"
date:       2015-04-09
catalog: false
tags:
    - C
---

对象指针可以被转化为`UnsafePointer<Void>`，同时也可以被转化回来。在OC中，可能是这样

```ObjC
void *voidPtr = (__bridge void*)self;
MyType *mySelf = (__bridge MyType*)voidPtr;

```

Swift中有个`Unmanaged`可以完成这个需求。因为它是使用`COpaquePointer`来替代`UnsafePointer<Void>`的，所以使用起来可能有些麻烦。

```Swift
func bridge<T: AnyObject>(obj: T) -> UnsafePointer<Void> {
	return UnsafePointer(Unmanaged.passUnretained(obj).toOpaque())
}

func bridge<T: AnyObject>(prt: UnsafePointer<Void>) -> T {
	return Unmanaged<T>.fromOpaque(COpaquePointer(ptr).takeUnretainedValue()
}
```

这种复杂的表达只是为了满足Swift严格的类型系统。在编译时，这仅仅是为了转换两个指针的类型而已。也可以使用将self传递进来

```Swift
let voidPtr = bridge(self)
```

或者使用`UnsafeMutablePointer<Void>(bridge(self)).

也可以转换回对象指针

```Swift
let mySelf: MyType = bridge(voidPtr)
```

另外，OC中的`__bridge_retained`和`__bridge_transfer`，在Swift中也可以用下面的代码表示

```Swift
func bridgeRetained<T: AnyObject>(obj: T) -> UnsafePointer<Void> {
	return UnsafePointer(Unmanaged.passRetained(obj).toOpaque())
}

func bridgeTransfer<T: AnyObject>(ptr: UnsafePointer<Void>) -> UnsafePointer<Void> {
	return Unmanaged<T>.fromOpaque(COpaquePointer(ptr)).takeRetainedValue()
}
```


原文链接：[How to cast self to UnsafeMutablePointer<Void> type in swift](http://stackoverflow.com/questions/33294620/how-to-cast-self-to-unsafemutablepointervoid-type-in-swift)
