---
layout:     post
title:      "Swift - UIRefresh"
subtitle:   " UIRefresh"
date:       2016-06-13
catalog: true
tags:
    - Swift
    - UIRefreshControl
---

UIRefreshControl通常用于`UITableView`和`UICollectionView`的下拉刷新。一个`UIRefreshControl`控件通常可以修改其`tintColor`和`attributedTitle`的值。

一开始`UIRefreshControl`初始化完成后，想要添加到`tableView`的时候，总想着`tableView`应该有个`setRefreshControl()`方法可以用来设置这个控件。最后查阅资料的时候才发现，要将刷新控件添加到`tableView`的话，只需要调用`addSubview`方法，然后添加Value Changed的Target-Action即可

```swift
...
var refreshControl: UIRefresh?

...

refreshControl = UIRefreshControl()
refreshControl!.tintColor = UIColor.whiteColor()
refreshControl!.attributedTitle = NSAttributedString(string: String(NSDate()), attributes: [NSForegroundColorAttributeName: UIColor.whiteColor()])
refreshControl!.addTarget(self, action: #selector(ViewController.refresh), forControlEvents: .ValueChanged)
tableView.addSubview(refreshControl!)
```

如果是自定义的UIRefreshControl的话，可以利用xib或者用代码的文件，然后将该view作为UIRefreshControl的子视图。可以参考[iOS Custom Pull-to-Refresh Control Tutorial (Updated for Swift)](http://www.jackrabbitmobile.com/design/ios-custom-pull-to-refresh-control/)
