---
layout:     post
title:      "利用AVFoundation获取视频缩略图"
date:       2016-05-24
author:     "Sim"
catalog: false
tags:
    - AVFoundation
---

利用AVFoundation获取视频缩略图的代码如下

```swift
func getVideoThumb(videoURL: String) -> UIImage {
    let url:NSURL = NSURL(string: videoURL)!
    let asset:AVURLAsset = AVURLAsset(URL:url, options: nil)
    let gen :AVAssetImageGenerator = AVAssetImageGenerator(asset: asset)
    gen.appliesPreferredTrackTransform = true

    let time: CMTime = CMTimeMakeWithSeconds(0.0, 600)
    var actualTime:CMTime = CMTime()

    let image:CGImageRef = try! gen.copyCGImageAtTime(time, actualTime: &actualTime)
    let thumb:UIImage = UIImage(CGImage: image)

    return thumb;
  }

```
