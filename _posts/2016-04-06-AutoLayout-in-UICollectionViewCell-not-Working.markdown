---
layout:     post
title:      "UICollectionViewCell布局约束失效解决方法"
date:       2016-04-06
author:     "Sim"
catalog: true
tags:
    - UICollectionViewCell
    - Bugs
---

利用Storyboard来设置UICollectionViewCell的话，一开始总是发现，给图片设置的约束总是失效。在Stack Overflow上找到这个问题[AutoLayout in UICollectionViewCell not working](http://stackoverflow.com/questions/25804588/auto-layout-in-uicollectionviewcell-not-working?rq=1),才知道一个bug。但是上面描述的是iOS 8 SDK在iOS 7设备上出现的错误。但是我使用的是Xcode 7.3和iOS 9.3的模拟器系统，但还是会出现同样的错误。所以就记录一下解决方法

ObjC版本

```objc
- (void)setBounds:(CGRect)bounds {
	[super setBounds:bounds];
	self.contentView.frame = bounds;
}
```

Swift版本

```swift
override var bounds: CGRect {
	didSet {
		contentView.frame = bounds
	}
}
```
