---
layout:     post
title:      "MoviePlayer开发笔记"
date:       2016-05-20
author:     "Sim"
catalog: true
tags:
    - FFMPEG
    - VideoToolbox
    - OpenGL ES
---

# 基础知识

# FFMPEG

FFMPEG是个强大的多平台通用的媒体文件的C语言编写的框架。iOS上较为有名的kxmovie，B站开源的播放器ijkplayer，据说国内其他视频平台比方说爱奇艺，优酷等的播放器，都是利用FFMEPG进行音频和视频的处理。FFMPEG可以通过命令行转换文件格式，处理各种格式的音频、视频文件。真的是用过的都说好。

## 在iOS上进行FFMPEG的使用

1. 安装yasm. 要将FFMPEG编译成iOS可以使用的框架的话，首先在Mac上安装yasm（应该没有人先安装homebrew吧）。使用`brew install yasm`就可以启动安装，完成以后可以用`yasm --version`来检测是否安装成功

2. 在Github上下载gas-preprrocessor: [下载传送门](https://github.com/applexiaohao/gas-preprocessor).下载完成以后，将gas-preprrocessor.pl文件copy到/usr/sbin/目录下，修改权限为777. `chmod 777 gas-preprrocessor.pl`

3. 在Github上clone FFmpeg-iOS-build-script. `git clone https://github.com/kewlbear/FFmpeg-iOS-build-script.git` 然后执行`build-ffmpeg.sh`文件。

  编译所有版本的arm64, armv7, x86_64静态库: `./build-ffmpeg.sh`

  编译arm64静态库: `./build-ffmpeg.sh arm64`

  编译arm7v和x86-64静态库: `./build-ffmpeg.sh armv7 x86_64`

  编译合并的版本: `./build-ffmpeg.sh lipo`

4. 使用编译生成的静态库。将FFMPEG-iOS整个文件夹添加到项目中，添加依赖库libz.tbd, libbz2.tbd, libiconv.tbd. 如果出现编译错误的话，在Build Setting - Header Search Paths中，添加FFMPEG-iOS/include即可（将整个文件夹拉进去即可）

5. 主要框架

  * libavutil -- 包含了简化编程的函数方法，诸如随机数生成器，帧的数据结构，还有一些数学工具等。

  * libavcodec -- 音频/视频的编解码工具

  * libavformat -- demuxer和muxer多媒体容器格式

  * libavdevide -- 用于获取输入输出的设备的软件框架

  * libavfilter -- 媒体过滤器，是一个用于处理画面的工具。视频方面的话，可以用于模糊，锐化，颜色处理，logo添加等工作。音频方面的话我还不知道能怎么用

  * libswscale -- 图片格式处理。常见于转换像素格式，比方说YUV420P转换为RGB

  * libswresample -- 音频采样工具

6. 常见用法（代码）

  1) 初始化

  ```objc
  avcodec_register_all(); // 注册所有的编解码器
  av_register_all(); // 初始化libavformat,muxers, demuxers和其他协议
  avformat_network_init(); // 如果使用的是网络媒体流文件的话，初始化network
  ```

  2）打开文件

  ```objc
  // 通常一些rtsp文件会有选项，如果是rtsp文件流的话, 添加下面两行代码
  AVDictionary *opts = 0;
  av_dict_set(&opts, "rstp_transport", "tcp", 0);

  // 打开影片档案(在头文件中声明AVFormatContext *pFormatCtx, filePath为文件地址)
  if (avformat_open_input(&pFormatCtx, [filePath UTF8String], NULL, &opts) < 0) {
    NSLog(@"Couldn't open file");
  }
  ```

  3) 获取文件流信息

  ```objc
  if (avformat_find_stream_info(pFormatCtx, NULL) < 0) {
    NSLog(@"Couldn't find stream info");
  }

  // 通常一个movie文件会包含有音频，视频，甚至可能会有字幕文件（基本套路都差不多的）
  // 声明为全局对象会好点
  int videoStream = -1;
  int audioStream = -1;
  for (int i = 0; i < pFormatCtx->nb_streams; i++) {
    if (pFormatCtx->streams[i]->codec->codec_type == AVMEDIA_TYPE_VIDEO) {
      videoStream = i;
    }

    if (pFormatCtx->streams[i]->codec->codec_type == AVMEDIA_TYPE_AUDIO) {
      audioStream = i;
    }
  }

  // 如果stream为-1的话，说明没有找到相应的流信息
  if (videoStream == -1) {
    NSLog(@"Couldn't find video stream");
  }

  if (audioStream == -1) {
    NSLog(@"Couldn't find audio stream");
  }

  // 都没找到的话，就可以退出这个流程了。
  // avformat_free_context(pFormatCtx); // 释放掉对象

  // AVCodecContext *pVideoCodecCtx;
  // AVCodecContext *pAudioCodecCtx;
  // 上面两个对象可以声明为全局的
  pVideoCodecCtx = pFormatCtx->stream[videoStream]->codec;
  pAudioCodecCtx = pFormatCtx->stream[audioStream]->codec;
  ```

  4) 获取并开启解码器

  ```objc
  AVCodec codec = avcodec_find_decoder(pVideoCodecCtx->codec_id);
  if (!codec) {
    NSLog(@"No video decoder");
    avcodec_free_context(pVideoCodecCtx);
    return;
  }

  // 开启解码器
  if (avcodec_open2(pVideoCodecCtx, codec, NULL) < 0) {
    NSLog(@"Couldn't open video decoder");
    avcodec_free_context(pVideoCodecCtx);
    return;
  }

  // AVFrame *pFrame;
  pFrame = av_frame_alloc();
  ```

  5) 解码开始

  ```objc
  AVPacket packet;
  while (av_read_frame(pFormatCtx, &packet) >= 0) {
    // 判断是否为视频
    if (packet.stream_index == videoStream) {
      int gotFrame = 0;
      int len = avcodec_decode_video2(pVideoCodecCtx, pFrame, &gotFrame, &packet);
      if (gotFrame) {
        // 在这里就已经解析出每帧画面pFrame的信息，根据需要进行处理，像kxmovie是创建了KxVideoFrameYUV对象，当然了。也可以转化为RGB图像进行输出
      }
    }
  }
  ```

  需要注意的一点是，解码和开启解码器的过程有可能会阻塞线程，所以最好使用异步线程进行这两个步骤的处理

7. 补充用法

  1) YUV转RGB

  ```objc
  // 声明对象
  struct SwsContext img_convert_ctx;
  AVFrame *pFrameRGB;

  pFrame = av_frame_alloc();
  pFrameRGB = av_frame_alloc();
  uint8_t* buffer = NULL;
  int numBytes = av_image_get_buffer_size(AV_PIX_FMT_YUV420P, pVideoCodecCtx->width, pVideoCodecCtx->height, 32); // YUV420P为源格式，其他格式是的视频的话，进行替换即可
  buffer = (uint8_t*)av_malloc(numBytes*sizeof(uint8_t));
  av_image_fill_arrays(pFrameRGB->data, pFrameRGB->linesize, NULL, AV_PIX_FMT_RGB24, pVideoCodecCtx->width, pVideoCodecCtx->height, 1);

  // 初始化img_convert_ctx(只进行一次)
  if (!img_convert_ctx) {
    img_convert_ctx = sws_getContext(pVideoCodecCtx->width, pVideoCodecCtx->height, pVideoCodecCtx->pix_fmt, pVideoCodecCtx->width, pVideoCodecCtx->height, AV_PIX_FMT_RGB24, SWS_BILINEAR, NULL, NULL, NULL);
  }

  // 在解码出帧数据后
  sws_scale(img_convert_ctx, (uint8_t const * const *)pFrame->data, pFrame->linesize, 0, pVideoCodecCtx->height, pFrameRGB->data, pFrameRGB->linesize)
  ```

  2) 输出UIImage

  ```objc
  - (UIImage *)imageFromAVPicture:(AVFrame)pict width:(int)width height:(int)height {
    CGBitmapInfo bitmapInfo = kCGBitmapByteOrderDefault;
    CFDataRef data = CFDataCreateWithBytesNoCopy(kCFAllocatorDefault, pict.data[0], pict.linesize[0]*height,kCFAllocatorNull);
    CGDataProviderRef provider = CGDataProviderCreateWithCFData(data);
    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
    CGImageRef cgImage = CGImageCreate(width,
                                       height,
                                       8,
                                       24,
                                       pict.linesize[0],
                                       colorSpace,
                                       bitmapInfo,
                                       provider,
                                       NULL,
                                       NO,
                                       kCGRenderingIntentDefault);
    CGColorSpaceRelease(colorSpace);
    UIImage *image = [UIImage imageWithCGImage:cgImage];
    CGImageRelease(cgImage);
    CGDataProviderRelease(provider);
    CFRelease(data);

    return image;
  }
  ```

  3) fps

  ```c
  static void avStreamFPSTimeBase(AVStream *st, CGFloat defaultTimeBase, CGFloat *pFPS, CGFloat *pTimeBase)
  {
    CGFloat fps, timebase;

    if (st->time_base.den && st->time_base.num)
      timebase = av_q2d(st->time_base);
    else if(st->codec->time_base.den && st->codec->time_base.num)
      timebase = av_q2d(st->codec->time_base);
    else
      timebase = defaultTimeBase;

    if (st->codec->ticks_per_frame != 1) {
      //timebase *= st->codec->ticks_per_frame;
    }

    if (st->avg_frame_rate.den && st->avg_frame_rate.num)
      fps = av_q2d(st->avg_frame_rate);
    else if (st->r_frame_rate.den && st->r_frame_rate.num)
      fps = av_q2d(st->r_frame_rate);
    else
      fps = 1.0 / timebase;

    if (pFPS)
      *pFPS = fps;
    if (pTimeBase)
      *pTimeBase = timebase;
  }
  ```

FFMPEG的大概用法就有上面的代码，其中模糊，锐化等我只看过少量的代码，并没有深入理解。所以这里就不说了。FFMPEG利用的是软解码的方式对流媒体文件进行编解码，所以在一定程度上会占用比较多的内存。Apple有提供VideoToolbox.framework用于硬编码，但是我在编写硬解码的代码时，总是出现bug，而且也不知道错误在哪里。等有时间再研究研究，关于VideoToolbox的介绍的话，大家可以移步我的[简书](http://www.jianshu.com/p/6dfe49b5dab8)查看

我在编写过程中，使用到的关于FFMPEG的内容比较少，目前来说，主要是使用FFMPEG来对H.264码流进行解码，关于流文件的时间等还没接触到，等接触到了再进行文章的更新。

# VideoToolbox

# OpenGL ES

# 开发过程要点记录

1.  要使用FFMPEG、VideoToolbox和OpenGL ES的话，需要导入以下框架：`CoreMedia.framework`、`CoreVideo.framework`、`libz.tbd`、`libbz2.tbd`、`libiconv.tbd`、`OpenGLES.framework`、 `VideoToolbox.framework`. 可选性导入: `CoreAudio.framework`、`AudioToolbox.framework`

2. 在使用OpenGL函数时，出现return 0的情况，原因在于当前的context没有初始化或者是没有设置。

3. 切记。。UI的改变要在主线程中进行调试。。。浪费了一天时间在测。。

4. 使用OpenGL在渲染平面图像的纹理时，比方说使用kxmovie播放的H264码流文件，会发现画面是比较模糊的。而B站开源的播放器播放出来就很清晰。把两者的代码对比了很久，也没找到是什么原因。后来在放大和缩小模拟器时发现，模拟器缩小的时候画面就变得很清晰？？？。。于是就考虑是不是分辨率的问题。于是发现了b站的播放器中设置了layer的scale。。。就是这么简单的代码我花了两三天的时间去找。。。
