---
layout:     post
title:      "Swift中的Lock"
date:       2016-06-22
author:     "Nov.柒月"
catalog: false
tags:
    - Synchronized
    - Lock
    - Thread
---

最近在工作上对于并发处理的代码越来越多。一般情况下，在iOS开发上，线程的处理中通常是使用GCD，NSOperation或者是NSThread. 这三者的封装从高到低，GCD是最经常使用的，也是Apple官方所推荐的。不过这三者我还没仔细研究，等以后有时间再仔细看看。

今天需要的一个需求是避免两个线程访问同一个变量，导致变量变化。也就是说给某个变量加上互斥锁。

在OC中，要在不同线程中安全访问同个资源的话，可以使用`@synchronized`关键字来加锁解锁

```objc
- (void)methodName:(id)anObj {
  @synchronized(anObj) {
    // 在这对anObj进行处理，anObj不会被其他线程所改变
  }
}
```

然而，虽然知道这个做法，但是，在swift中，发现没有了synchronize这个关键字。Google了一下，找到喵神的帖子。其做法是写一个全局方法，接受闭包来实现synchronize的功能。

```swift
func synchronized(lock: AnyObject, closure:()->()) {
  objc_sync_enter(lock)
  closure()
  objc_sync_exit(lock)
}
```

因为根据喵神所讲，`@synchronized`实际上是调用了`objc_sync`的两个方法并且加入了异常的判断。上面的这种做法实际上是去除了异常的判断的简单版本。不过在swift中如果需求不算很严格的话，这种做法还是能够避免不同线程访问同一资源造成的异常的现象。

---

## 传送门

[喵神 - Lock](http://swifter.tips/lock/)
