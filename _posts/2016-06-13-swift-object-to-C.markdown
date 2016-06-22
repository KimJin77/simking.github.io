---
layout:     post
title:      "Swift对象转C指针"
subtitle:   "Swift Object in C"
date:       2016-06-13
catalog: true
tags:
    - Swift
    - C
---

```swift
class Bridge {
  // 前两个是使用__bridge
  class func bridge<T: AnyObject>(obj: T) -> UnsafePointer<Void> {
    return UnsafePointer(Unmanaged.passUnretained(obj).toOpaque())
  }

  class func bridge<T: AnyObject>(ptr: UnsafePointer<Void>) -> T {
    return Unmanaged<T>.fromOpaque(COpaquePointer(ptr)).takeUnretainedValue()
  }

  // 后两个是__bridge_transfer
  class func bridgeRetained<T : AnyObject>(obj : T) -> UnsafePointer<Void> {
    return UnsafePointer(Unmanaged.passRetained(obj).toOpaque())
  }

  class func bridgeTransfer<T : AnyObject>(ptr : UnsafePointer<Void>) -> T {
    return Unmanaged<T>.fromOpaque(COpaquePointer(ptr)).takeRetainedValue()
  }
}

// 转UnsafeMutablePointer的话
UnsafeMutablePointer(Unmanaged.passUnretained(self).toOpaque())
```
