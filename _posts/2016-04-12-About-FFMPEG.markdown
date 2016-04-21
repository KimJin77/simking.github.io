---
layout:     post
title:      "FFMPEG学习"
date:       2015-04-12 12:00:00
author:     "Sim"
catalog: true
tags:
    - FFMPEG
---

# 基础知识

* Stream: 流。可以理解为一段时间内可接收的数据元素。比方说视频流和音频流
* Frame: 帧。流里面的数据元素称之为帧。不同的流有不同类型的编解码器。编码器决定了数据的处理方式。
* Packet: 包。从流中获取包。包由可以解码成原始帧的数据段组成，且最后用于App操作。

# 操作步骤

1）从媒体文件中打开视频流

2）将从视频流中读取到的包转换为帧

3）如果帧数据不完整，继续步骤2

4）处理帧数据

5）回到步骤2

## 具体操作

1. 通常需要导入下面三个FFMPEG的头文件

```ObjC
#import "libavcodec/avcodec.h"
#import "libavformat/avformat.h"
#import "ffmpeg/swscale.h"
```

对于iOS开发的话，需要添加下列依赖库

* libiconv.tbd
* libz.tbd
* libbz2.tbd
* CoreMedia.framework
* AudioToolbox.framework
* VideoToolbox.framework

2. 在使用FFMPEG之前，需要事先调用`av_register_all()`方法来注册文件格式和编解码器。`av_register_all()在整个app的生命周期中只需要调用一次。


3. 打开文件

```ObjC
AVFormatContext *pFormatCtx;

// Open video file
if (avformat_open_input(&pFormatCtx, [path UTF8String], NULL, NULL) != 0) {
	NSLog(@"Couldn't open file);
}
```

该方法将文件头和相关信息存储在`AVFormatContext`结构的对象中。最后的三个参数分别用于指定文件格式，缓存大小和格式选项，可以设置为`NULL`或者0，因为libavformat会自动检查这三个参数的值。

4. 获取流信息, 在`pFormatCtx->steams`中填入合适的信息。

```ObjC
if (avformat_find_stream_info(pFormatCtx, NULL) < 0 ) {
	NSLog(@"Couldn't find stream information");
}
```

可以通过下面的debug函数来获取信息

```ObjC
av_dump_format(pFormatCtx, 0, [path.lastPathComponent UTF8String], false);
```

5. 遍历`pFormatCtx->streams`来获取视频流

```ObjC
int videoStream = -1;

for (int i = 0; i < pFormatCtx->nb_streams;i++) {
	// 音频流的话,codec_type为AVMEDIA_TYPE_AUDIO
	if (pFormatCtx->streams[i]->codec->codec_type == AVMEDIA_TYPE_VIDEO) {
		videoStream = i;
		break;
	}
}

if (videoStream == -1) {
	NSLog(@"Didn't find a video stream");
	return;
}

// 获取视频流的编解码器的上下文的指针
pCodecCtx = pFormatCtx->streams[videoStream]->codec;
```

流信息中，关于编解码器的信息我们称之为"编解码器上下文".里面包含了流所使用的编解码器的所有信息。虽然如此，但是我们仍然需要找到编解码器并打开之

6. 获取编解码器

```ObjC
AVCodec *pCodec;

// Find the decoder
pCodec = avcodec_find_decoder(pCodecCtx->codec_id);
if (pCodec == NULL) {
	NSLog(@"Unsupported codec");
	return;
}

// Copy
pCodecCtx = avcodec_alloc_context3(pCodec);
if (avcodec_copy_context(pCodecCtx, pCodecCtxOrig) != 0) {
	NSLog(@"Couldn't copy codec context");
	return;
}

// Open
if (avcodec_open2(pCodecCtx, pCodec) < 0) {
	NSLog(@"Couldn't open codec);
	return;
}
```

**Note** 不能直接使用视频流的AVCodecContext！所以使用`avcodec_copy_context()`来复制一份。

7. 存储数据

在内存中存储帧数据

```ObjC
AVFrame *pFrame;

// Allocate
pFrame = av_frame_alloc();
```

假设我们想要输出以24位RGB保存数据的PPM文件，我们需要将帧从原始的数据转化为RGB。FFMPEG能帮我们完成这个功能滴。如：

```ObjC
// Allocate an AVFrame structure
pFrameRGB = av_frame_alloc();
if (pFrameRGB == NULL) return;
```

分配内存后，在我们转换数据的时候，需要存储原始数据的地方。我们可以使用`avpicture_get_size`来获取大小并且分配空间

```ObjC
uint8_t *buffer = NULL;
int numBytes;
// 获取所需要的内存缓存大小并分配空间
numBytes = avpicture_get_size(PIX_FMT_RGB24, pCodecCtx->width, pCodexCtx->height);
buffer = (uint8_t *)av_malloc(numBytes * sizeof(uint8_t));

avpicture_fill((AVPicture *)pFrameRGB, buffer, PIX_FMT_RGB24, pCodecCtx->width, pCodecCtx->height);
```

9. 读取数据

通过读取包来获取整个完整的视频流，然后进行解码，一旦完成所需要的帧数据的读取，就可以进行转换并保存了。

```ObjC
struct SwsContext *sws_ctx;
int frameFinished;
AVPacket packet;

// 初始化SWS上下文
sws_ctx = sws_getContext(pCodecCtx->width, pCodecCtx->height, pCodecCtx->pix_fmt,pCodecCtx->width, pCodecCtx->height, PIX_FMT_RGB24, SWS_BILINEAR, NULL,NULL, NULL);

while(av_read_frame(pFormatCtx, &packet) >= 0) {
	// 判断是否来自视频流
	if (packet.stream_index == videoStream) {
		// 解码
		avcodec_decode_video2(pCodeCtx, pFrame, &frameFinished, &packet);
	}

	// 是否获取到视频帧
	if (frameFinished) {
		// 转换为RGB
		sws_scale(sws_ctx, (uint8_t const * const *)pFrame->data, pFrame->linesize, 0, pCodecCtx->height, pFrameRGB->data, pFrameRGB->linesize);

		// 保存
		if (++i <= 5) {
			SaveFrame(pFrameRGB, pCodecCtx->width, pCodecCtx->height, i);
		}
	}

	// 释放
	av_free_packet(&packet)
}
```

首先，使用`av_read_frame()`读取包，并保存到`AVPacket`结构中。我们只需要分配包结构的空间，FFMPEG会自行分配内部的数据结构。最后使用`av_free_packet()`释放。`avcodec_decode_video()`负责将包转换成帧。但是，因为我们不知道解码所需要的全部的数据，所以在有下一帧数据的时候通过`avcodec_decode_video()`来设置frameFinished。使用`sws_scale()`来转换数据.

保存的方法

```ObjC
FILE *pFile;
NSString *fileName;
int y;

NSString *documentsDirectory = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
fileName = [documentsDirectory stringByAppendingPathComponent:fileName];

// Open file
pFile = fOpen([fileName UTF8String], "wb");
if (pFile == NULL) return;

// write header
fprintf(fFile, "P6\n%d %d\n255\n", width, height);

// write pixel data
for (y = 0; y < height; y++) {
	fwrite(pFrame->data[0]+y*pFrame->linesize[0], 1, width*3, pFile);
}

// close file
fclose(pFile);
```

10. 释放

```ObjC
av_free(buffer);
av_free(pFrameRGB);

av_free(pFrame);

avcodec_close(pCodecCtx);
avcodec_close(pCodecCtxOrig);

avformat_close_input(&pFormatCtx);
```

# 在屏幕上播放

iOS好像使用OpenGL比较多，有待研究。FFMPEG的教程中使用的是SDL。

# 播放声音

iOS上使用AudioStreamer

<!--
# 视频同步

## PTS和DTS

在声音和视频都能正常播放之后，就该考虑如何同步播放。不然就是各播各的了。音频流和视频流里面都包含有播放速率等相关信息。音频流有采样率，视频流有帧率。但是，视频和音频同步并不是简单的数帧数和采样帧率就能够完成的。流数据中的包里面，有两个属性，分别是解码时间标记(Decoding time stamp -- DTS)和播出时间标记(presentation time stamp -- PTS).要理解这两者的话，需要知道媒体文件的存储方式。比方说MPEG，使用的是"B"帧(B代表的是bidirectional,双向的)。另外的I和P代表的是帧内(Intra)和预测(Predicted).I帧包含了完整的图片，P帧取决于先前的I帧和P帧，并且不同之处在于增量方面。B帧和P帧差不多，但是取决于某帧画面之前或之后的信息。

假设我们有个电影文件，每帧画面是I B B P。我们需要在播放每个B帧之前知晓P中的信息。由于这个原因，所以这个文件的帧可能按I P B B的顺序进行存储。这就是为什么我们要在每帧画面上用上DTS和PTS。DTS告诉我们什么时候进行解码，PTS则会告诉我们什么时候可以开始播放。所以，我们的帧看起来是这样的

---
	PTS: 1 4 2 3
	DTS: 1 2 3 4
Stream: I P B B
---

通常情况下，PTS和DTS只会在播放B帧的时候出现不同。

当我们从`av_read_frame()`中获取包时，将会得到包中的PTS和DTS的值。但是真正需要的是我们最新解码出来的帧的PTS，因为我们需要知道什么时候能够进行播放。FFMPEG提供了`av_frame_get_best_effort_timestamp()`函数

## 同步

在播放一帧画面后，我们需要知道下一帧画面的播出时间。所以我们可以简单地设置一个计时器来刷新视频画面。如果是这样的话，就要对比下一帧的PTS和系统时间来确定计时器的设置时长。同时还要考虑下面两点：

首先是知道下一帧的PTS的时间。虽然我们知道可以将当前的视频速率添加到当前的PTS上，但是有些视频是需要重复帧画面来完成的。也就是说，我们可以在一段时间里面重复播放当前的画面。这可能会引起下一帧画面的过快出现。

我们可能不会过于关心是不是每帧画面和声音都能完美同步。因为不是每个媒体文件都是无缺陷的，所以我们可以将音频同步到视频，将视频同步到音频，甚至是将这两者同步到一个外部时钟（比方说你的电脑）.

## 获取某帧画面的PTS

使用`avcodec_decode_video2`

```ObjC
double pts;

for (;;) {
	...

	pts = 0;
	// 解码
	len1 = avcodec_decode_video2(videoCodecCtx, videoFrame, &frameFinished, packet);

	if (packet->dts != AV_NOPTS_VALUE) {
		pts = av_frame_get_best_effort_timestamp(pFrame);
	} else {
		pts = 0;
	}

	pts *= av_q2d(fFormatCtx->stream[videoStream]->time_base);
}
```

## 同步，使用PTS

（下面的代码使用C，我还没在App中用过，所以照搬教程）

```C
typedef struct VideoState {
	double video_clock; // 最后解码帧画面的pts/下一帧解码的pts
```

```C
double synnchronize_video(VideoState *is, AVFrame *src_frame, double pts) {
	double frame_delay;

	if (pts != 0) {
		// 如果pts不为0，设置为video_clock
		is->video_clock = pts;
	} else {
		// 如果没获取到pts，就将其设置为clock
		pts = is->video_clock;
	}

	// 更新clock
	frame_delay = av_q2d(is->video_st->codec->time_base);
	// 如果画面重复，则根据clock进行调整
	frame_delay += src_frame->repeat_pict * (frame_delay * 0.5);
	is->video_clock += frame_delay;
	return pts;
}
```

使用queue_picture来使用PTS，将帧添加进队列

```C
if (frameFinished) {
	pts = synchronize_video(is, Frame, pts);
	if (queue_picture(is, pFrame, pts) < 0) {
		break;
	}
}
```

我们通过简单的计算前一个pts和当前这个pts的时间来获取下一个pts。与此同时，我们需要将视频同步到音频上面去。创建一个音频时钟，用于追踪当前音频播放的位置。

现在先假设我们有个`get_audio_clock`函数可以获取音频时钟的时间。不过，在视频和音频没同步的时候，我们应该怎么使用这个值呢？我们可以使用计算下一次刷新的时间来调整这个时间值。如果PTS相比音频时间较后的话，我们可加大延迟时间。如果太过超前，则可以加快刷新频率。无论是延迟还是加快，我们都需要通过`frame_timer`来计算。这个帧计时器会在播放影片的时候统计所有的延迟时间。换句话说，`frame_timer`就是下一帧画面要播出的时间。

```ObjC
void video_refresh_timer(void *userdata) {
	VideoState *is = (VideoState *)userdata;
	VideoPicture *vp;
	double actual_delay, delay, sync_threshold, ref_clock, diff;

	if (is->video_st) {
		if (is->pictq_size == 0) {
			schedule_refresh(is, 1);
		} else {
			vp = &is->pictq[is->pictq_rindex];

			delay = vp->pts - is->frame_last_pts; // 最后时间的pts
			if (delay <= 0 || delay >= 1.0) {
				// 延迟的时间不对的话，使用前一个
				delay = is->frame_last_delay;
			}

			// 保存
			is->frame_last_delay = delay;
			is->frame_last_pts = vp->pts;

			// 更新
			ref_clock = get_audio_clock(is);
			diff = vp->pts - ref_clock;

			// 跳过或者重复某帧画面
			sync_threshold = (delay > AV_SYNC_THRESHOLD) ? delay: AV_SYNC_THRESHOLD;
			if (fabs(diff) < AV_NOSYNC_THRESHOLD) {
				if (diff <= -sync_threshold) {
					delay = 0;
				} else if (diff >= sync_threshold) {
					delay = 2 * delay;
				}
			}
			is->frame_timer += delay;
			// 计算实际延迟
			actual_delay = is->frame_timer - (av_gettime() / 1000000.0);
			if (actual_delay < 0.010) {
				actual_delay = 0.010;
			}

			schedule_refresh(is, (int)(actual_delay * 1000 + 0.5));
			video_display(is);

			// 更新队列
			if (++is->pictq_rindex == VIDEO_PICTURE_QUEUE_SIZE) {
				is->pictq_rindex = 0;
			}
			SDL_LockMutex(is->pictq_mutex);
			is->pictq_size--;
			SDL_CondSignal(is->pictq_cond);
			SDL_UnlockMutex(is->pictq_mutex);
		}
	} else {
		schedule_refresh(is, 100);
	}
}
```
-->

---
相关链接:[An ffmpeg and SDL Tutorial](http://dranger.com/ffmpeg/tutorial01.html)
