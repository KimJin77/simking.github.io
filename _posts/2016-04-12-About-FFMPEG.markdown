---
layout:     post
title:      "FFMPEG学习贴"
date:       2015-05-05
author:     "Sim"
catalog: true
tags:
    - FFMPEG
---

# FFMPEG

FFMPEG是个强大的多平台通用框架，在目前我所能找到的代码中，几乎大部分都是使用FFMPEG来完成视频播放器。FFMPEG可以用于编解码，mux，demux，流数据处理，播放等功能。其包含的大类有：

  * libavutil -- 包含了简化编程的函数方法，比方说随机数生成器，数据结构，数学工具，核心媒体工具等。
  * libavcodec -- 音频/视频的编解码工具
  * libavformat -- 包含了demuxer和muxer的多媒体容器格式
  * libavdevice -- 包含了用于获取/渲染的输入/输出设备的软件框架，诸如Video4Linux, Video4Linux2, VfW和ALSA
  * libavfilter -- 媒体过滤器
  * libswscale -- 图片优化
  * libswresample -- 音频采样优化

通常视频播放器的话，使用libavcodec, libavformat, libswscale这三个头文件的比较多。

## 在iOS上使用

1. 安装yasm

  因为我的电脑已经安装了Homebrew，所以使用`brew install yasm`即可自动安装。安装完了以后，可以使用`yasm --version`来检测是否安装成功。

2. gas-preprrocessor

  在Github上下载gas-preprrocessor:[https://github.com/applexiaohao/gas-preprocessor](https://github.com/applexiaohao/gas-preprocessor)。下载完成以后，将gas-preprocessor.pl文件拷贝到/usr/sbin/目录下，修改权限为777.

3. FFmpeg-iOS-build-script

  Clone，然后执行`build-ffmpeg.sh`

  编译所有版本的arm64, armv7, x86_64静态库：`./build-ffmpeg.sh`

  编译arm64的静态库：`./build-ffmpeg.sh arm64`

  编译armv7v和x86-64的静态库：`./build-ffmpeg.sh armv7 x86_64`

  编译合并的版本: `./build-ffmpeg.sh lipo`


4. 使用编译完成的静态库

  将FFmpeg-iOS引入到Xcode，添加依赖库libz.tbd, libbz2.tbd, libiconv.tbd。如果出现编译错误的话，添加如下链接路径：Build Setting - Header Search Paths = $(SRCROOT)/FFMPEGDemo/FFmpeg-iOS/include

5. 最后根据项目需求添加其他依赖库，比方说:CoreAudio.framework, CoreMedia.framework, AudioToolbox.framework, VideoToolbox.framework等。

## 基础概念

* Stream: 流数据，即一连串可用的数据元素。举个例子来说，就是视频流和音频流。
* Frame: 帧，流中的数据元素，不同的帧需要不同的编解码器。
* Packet: 包，从流中获取的。包含了解码成原始帧的数据，可用于最后的处理。

## 解析

在我找到的demo中，比较有代表性的是[kxmovie](https://github.com/kolyvan/kxmovie)和B站的播放器[ijkplayer](https://github.com/Bilibili/ijkplayer)。因为看得比较多的是kxmovie，所以先学习下kxmovie的，B站的播放器先开个坑，等学习研究。

### kxmovie

#### 初始化

使用FFMPEG进行编解码操作的话，基本都是以`av_register_all()`开始。

`av_register_all()`用于初始化libavformat，注册所有的muxer，demuxer和其他相关的协议。通俗点讲，就是告诉系统“哎，这个文件的格式是可以打开的，它的解编码器是我们能用的”。

如果流数据不是本地文件的话，你还需要加上`avformat_network_init()`.

`avformat_network_init()`初始化了全局的网络组件。虽然是可选的，但是它隐式地避免了设置每个会话的开销。如果是网络文件的话，那么就是强制使用（有点废话的feel）

#### 开启文件

接下来就可以愉悦地打开文件了。我们看下KxMovieDecode中的`openInput:`方法：

```ObjC

- (kxMovieError) openInput: (NSString *) path
{
    AVFormatContext *formatCtx = NULL;

    // 1
    if (_interruptCallback) {

        formatCtx = avformat_alloc_context();
        if (!formatCtx)
            return kxMovieErrorOpenFile;

        AVIOInterruptCB cb = {interrupt_callback, (__bridge void *)(self)};
        formatCtx->interrupt_callback = cb;
    }

    // 2
    if (avformat_open_input(&formatCtx, [path cStringUsingEncoding: NSUTF8StringEncoding], NULL, NULL) < 0) {

        if (formatCtx)
            avformat_free_context(formatCtx);
        return kxMovieErrorOpenFile;
    }

    // 3
    if (avformat_find_stream_info(formatCtx, NULL) < 0) {

        avformat_close_input(&formatCtx);
        return kxMovieErrorStreamInfoNotFound;
    }
,
    // 4
    av_dump_format(formatCtx, 0, [path.lastPathComponent cStringUsingEncoding: NSUTF8StringEncoding], false);

    _formatCtx = formatCtx;
    return kxMovieErrorNone;
}

```

这段代码中，有个问题需要注意下。我试过在另外的Project中打下同样的代码，可是一执行到代码段1中就会crash，始终找不到原因。在stackflow上看到一个答案说是在执行`avformat_open_input()`的时候要确保传入的formatCtx为NULL。当我注释掉代码段1的时候，就发现能顺利编译运行过去了。。而kxmovie中的就算不注释，也能顺利运行。。真是无解。。不知道是不是使用的ffmpeg版本不同的原因。。。。。至今不明白。

`formatCtx`可以初始化为NULL，也可以如代码段1中那样调用`avformat_alloc_context()`方法进行初始化。（好像差别不大）。

比较重要的是`avformat_open_input()`方法。调用这个方法来开启输入流及读取头文件，但是并没有开启解编码器哦~`avformat_open_input()`接受4个参数：第一个是指向`AVFormatContext`的指针，第二个是文件的路径，需要使用UTF8编码，最后两个参数是文件格式和缓存大小等可设置的参数，基本都是传入`NULL`或者是0。因为libavformat会自动填充这些相关的信息。

#### 获取Stream

成功打开文件后，可以读取文件的流数据信息。

使用`avformat_find_stream_info()`方法来获取流信息，也就是给`formatCtx->streams`赋值。当返回值大于0的话，表明获取信息成功。则此时`formatCtx->streams`是一个size为`formatCtx->nb_streams`的数组，里面存的都是流数据的指针。

代码段4好像是我自己调试的时候加上的，主要就是用`av_dump_format()`方法来debug调试。

这段代码完成之后，我们就能获取到媒体文件的流数据。接下来的步骤就是进行流数据的处理了。

首先我们要获取流数据，通过遍历`formatCtx->streams`数组来获取视频/音频/字幕(subtitle在这里应该字幕吧)的index

```ObjC
// 这里只写videoStream

// 初始化为-1
NSInteger videoStream = -1;
for (NSInteger i = 0; i < formatCtx->nb_streams; i++) {
  if (formatCtx->streams[i]->codec->codec_type == AVMEDIA_TYPE_VIDEO) {
    videoStream = i;
    ...
  }
}

if (videoStream == -1) {
  return -1; // 没有找到视频流信息
}

// 音频的编解码器类型则是AVMEDIA_TYPE_AUDIO, 字幕是AVMEDIA_TYPE_SUBTITLE
```

#### 获取并打开解码器

流信息中关于编解码器的部分可以称为编解码器的上下文，里面包含了流所使用的编解码器的所有信息。接下来需要找到并打开编解码器。

```ObjC
// 获取编解码器上下文指针
AVCodecContext *codecCtx = _formatCtx->streams[videoStream]->codec;
// 找到解码器
AVCodec *codec = avcodec_find_decoder(codecCtx->codec_id);
if (codec == NULL) {
  NSLog("Unsupported codec!");
}
// 打开解码器
if (avcodec_open2(codecCtx, codec, NULL) < 0) {
  NSLog(@"Could not open codec.");
}
```

#### FPS

---
下面需要了解的是FFMPEG中的时间概念。

FFMPEG中大多数结构都有`time_base`的成员变量存在，比方说`AVStream`和`AVCodecContext`。`time_base`的结构如下:

```c
typedef struct AVRational {
  int num;
  int den;
} AVRational;

// AVRational是一个分数，其中num为分子，den为分母
```

`time_base`的单位是秒。

在FFMPEG中还有几个跟时间有关的概念

* fps(帧率) = st->avg_frame_rate

  `AVStream::avg_frame_rate` -- 平均帧率，在demuxing中，可能会在创建流数据时由libavformat或者在`avformat_find_stream_info()`中被赋值。


* tbr(AVStream中的time base) = st->r_frame_rate

  `AVStream::r_frame_rate` -- 流的真正帧速率。这是最底层帧速率，所有的时间戳都能使用其进行准确计算得出。但是！这个只是一个猜测值来的！！举个栗子，如果time base是1/90000,且所有的帧近似都是3600或者1800的计时器刻度的话，则`r_frame_rate`的值为50/1

* tbn(用于特殊要求的AVCodecContext的time base) = st->time_base

  `AVStream::time_base` -- 基本的时间单位（单位为秒），表示帧的时间戳。解码时由libavformat进行设置，编码时调用avformat_write_header()进行设置

* tbc(视频帧率) = st->codec->time_base

  `AVCodecContext::time_base` -- 基本时间单位（单位为秒），对于fixed-fps的文件而言，timebase应该是1/帧率并且时间戳的增量应该是1。tbr的精度要大于tbc

---

在kxmovie中，fps和timebase相关的代码段如下：

```ObjC
// av_q2d()函数用于把有理数转化成double类型的数

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
        LoggerStream(0, @"WARNING: st.codec.ticks_per_frame=%d", st->codec->ticks_per_frame);
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

我们可以看到，timebase是先从检测tbn的值，如果tbn为非0的话，则使用tbn作为timebase，为0就判断tbc，tbc为0就使用默认的timebase。fps也差不多，先判断fps，再判断tbr。

基本上，准备步骤就到了这里就完成了。接下来的步骤就是根据自己的需求灵活使用，FFMPEG的就这么完了？（这么坑）。从kxmovie中看到的，有下面这些

#### 视频帧格式

kxmovie中自定义了两种视频帧的格式：KxVideoFrameFormatYUV和KxVideoFrameFormatRGB。一般情况下，视频帧的格式也就这两种了。。我们可以通过`videoCodecCtx->pix_fmt`来获取视频帧的具体格式，对于YUV来说，其值可能是`AV_PIX_FMT_YUV420P`或者是`AV_PIX_FMT_YUVJ420P`(也有可能是其他YUV类型，kxmovie中只对这两种类型进行了判断)。

#### 解码（关键步骤来了）

首先判断所有帧是否已经读取完成，使用`av_read_frame()`。如果已经到了最后一帧的话，该函数会返回小于0的值
```ObjC
AVPacket packet;
while(av_read_frame(formatCtx, &packet) >= 0) {
  int gotFrame = 0;
  if (packet.stream_index == videoStream) {
    avcodec_decode_video2(codexCtx, frame, &gotFrame, &packet);
  }

  // 如果gotFrame为非0，则表明读取到了视频帧， 此时在kxmovie中，会根据视频帧的格式进行数据处理
  if (gotFrame) {
    [self handleVideoFrame];
  }
}

- (KxVideoFrame *) handleVideoFrame
{
    if (!_videoFrame->data[0])
        return nil;

    KxVideoFrame *frame;

    if (_videoFrameFormat == KxVideoFrameFormatYUV) {

        KxVideoFrameYUV * yuvFrame = [[KxVideoFrameYUV alloc] init];
        // 将读取到的数据分成Y、U、V分量
        yuvFrame.luma = copyFrameData(_videoFrame->data[0],
                                      _videoFrame->linesize[0],
                                      _videoCodecCtx->width,
                                      _videoCodecCtx->height);

        yuvFrame.chromaB = copyFrameData(_videoFrame->data[1],
                                         _videoFrame->linesize[1],
                                         _videoCodecCtx->width / 2,
                                         _videoCodecCtx->height / 2);

        yuvFrame.chromaR = copyFrameData(_videoFrame->data[2],
                                         _videoFrame->linesize[2],
                                         _videoCodecCtx->width / 2,
                                         _videoCodecCtx->height / 2);

        frame = yuvFrame;

    } else {
    // RGB格式的话
        if (!_swsContext &&
            ![self setupScaler]) {

            LoggerVideo(0, @"fail setup video scaler");
            return nil;
        }

        sws_scale(_swsContext,
                  (const uint8_t **)_videoFrame->data,
                  _videoFrame->linesize,
                  0,
                  _videoCodecCtx->height,
                  _picture.data,
                  _picture.linesize);


        KxVideoFrameRGB *rgbFrame = [[KxVideoFrameRGB alloc] init];

        rgbFrame.linesize = _picture.linesize[0];
        rgbFrame.rgb = [NSData dataWithBytes:_picture.data[0]
                                    length:rgbFrame.linesize * _videoCodecCtx->height];
        frame = rgbFrame;
    }    

    frame.width = _videoCodecCtx->width;
    frame.height = _videoCodecCtx->height;
    frame.position = av_frame_get_best_effort_timestamp(_videoFrame) * _videoTimeBase;

    const int64_t frameDuration = av_frame_get_pkt_duration(_videoFrame);
    if (frameDuration) {

        frame.duration = frameDuration * _videoTimeBase;
        frame.duration += _videoFrame->repeat_pict * _videoTimeBase * 0.5;

    } else {

        // sometimes, ffmpeg unable to determine a frame duration
        // as example yuvj420p stream from web camera
        frame.duration = 1.0 / _fps;
    }    
    return frame;
}

```
## 总结

完成上述步骤后，就可以利用OpenGL等进行渲染绘制了。

综上所述，利用FFMPEG进行流媒体文件的处理的话，有以下步骤

  1. 初始化libavformat库，初始化网络组件（可选）
  2. 打开媒体流文件 -- `avformat_open_input()`
  3. 获取流信息 -- `avformat_find_stream_info()`
  4. 遍历`formatCtx->streams`,根据编解码器类型判断流的类型 -- `formatCtx->stream[i]->codec->codec_type`
  5. 获取编解码器上下文指针 -- `AVCodecContext *codecCtx = formatCtx->streams[videoStream]->codec`
  6. 获取编解码器 -- `AVCodec *codec = avcodec_find_decoder(codecCtx->codec_id)`
  7. 打开编解码器 -- `avcodec_open2()`
  8. 设置fps和timebase
  9. 读取视频帧 -- `av_read_frame`
  10. 解码 -- `avcodec_decode_video2()`

---
相关链接:

1. [An ffmpeg and SDL Tutorial](http://dranger.com/ffmpeg/tutorial01.html)
2. [time-base in FFMPEG](https://gist.github.com/yohhoy/50ea5fe868a2b3695e19)
3. [kxmovie](https://github.com/kolyvan/kxmovie)
4. [YUV](https://en.wikipedia.org/wiki/YUV)
