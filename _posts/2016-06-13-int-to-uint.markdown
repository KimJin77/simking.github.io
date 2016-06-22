---
layout:     post
title:      "Swift - Int转UInt"
subtitle:   "Swift - Int to UInt"
date:       2016-06-13
catalog: false
tags:
    - Swift
    - Int
    - UInt
---

```swift
extension Int8 {
    init(_ v: UInt8)
    init(_ v: UInt16)
    init(truncatingBitPattern: UInt16)
    init(_ v: Int16)
    init(truncatingBitPattern: Int16)
    init(_ v: UInt32)
    init(truncatingBitPattern: UInt32)
    init(_ v: Int32)
    init(truncatingBitPattern: Int32)
    init(_ v: UInt64)
    init(truncatingBitPattern: UInt64)
    init(_ v: Int64)
    init(truncatingBitPattern: Int64)
    init(_ v: UInt)
    init(truncatingBitPattern: UInt)
    init(_ v: Int)
    init(truncatingBitPattern: Int)
    init(bitPattern: UInt8)
}
```
