---
layout:     post
title:      "OC中变量的下划线访问"
date:       2016-04-14
catalog: false
tags:
    - Objective-C
---

在学习OC开始，就在别人的Demo中看到，有些时候变量是用`_property`进行访问，有些时候使用`self.property`进行访问。一直也没有去细究过里面的问题。本着能用就用，并且苹果官方提倡的是使用`self`来对实例变量进行访问的原因，就一直没去研究。

今天在学习kxmovie的库的时候，就顺带搜了一下资料。下划线加变量名的话，只是直接设置了变量的值。而通过`self`来访问的话，是调用了getter和setter方法

e.g

```Objective-C
// ViewController.h
#import <UIKit/UIKit.h>

@interface ViewController: UIViewController
@property (nonatomic, strong) NSString *exampleString;
@end

// ViewController.m
@implementation ViewController
- (void)viewDidLoad {
  _string = @"Test";
  NSLog(@"%@", _exampleString);
  NSLog(@"%@", self.exampleString);
}
@end
```

上面的代码中，两个输出值都是Test

```Objective-C
// 如果设置了Getter
- (NSString *)exampleString {
  return @"Getter Method";
}
```

设置了Getter方法的话，第一个输出仍然是Test，但是第二个输出就会变成了Getter Method

```Objective-C
// 如果设置了Setter
- (void)setExampleString:(NSString *)exampleString {
  if ([_exampleString isEqualToString:@"Test"]) {
    _exampleString = @"Settter";
  }
}
```

设置了Setter的话，此时两个输出都会变成Setter。因为在setter方法中也修改了_exampleString的值。

所以，下划线加变量名对变量进行访问的话，只是简单的设置和读取值。使用self的话，是利用getter和setter进行访问的。

另外，如果设置了Getter方法的话，就不能使用下划线加变量名进行访问，Xcode会报`Use of undeclared identifier`的错误。可能因为这个原因，所以比较久以前的代码中，能看到作者在.h文件中生命了私有的下划线变量，又声明了property的实例变量。
