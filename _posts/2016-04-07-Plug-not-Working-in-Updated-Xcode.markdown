---
layout:     post
title:      "Xcode升级后插件失效"
date:       2016-04-07
author:     "Sim"
catalog: false
tags:
    - Xcode
---

每次Xcode升级都都会导致之前使用的插件没办法使用。主要原因是新版本的Xcode的UUID没有注册到插件的配置文件中。

之前通过手动修改的话感觉比较麻烦，所以就记录下命令行修改的方法

1. 找到Xcode的UUID

`defaults read /Applications/Xcode.app/Contents/Info.plist DVTPlugInCompatibilityUUID`

2. 插入UUID

`find /Users/{用户名}/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins -name Info.plist -maxdepth 3| xargs -I{} defaults write {} DVTPlugInCompatibilityUUIDs -array-add {UUID}`
