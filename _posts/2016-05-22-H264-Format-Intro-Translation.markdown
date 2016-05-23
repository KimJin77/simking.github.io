---
layout:     post
title:      "H.264介绍（从Stack Overflow答案翻译）"
date:       2016-05-22
author:     "Sim"
catalog: true
tags:
    - VideoToolbox
    - H.264
---

最近在研究如何利用iOS的VideoToolbox的框架对在线视频进行硬解码，从而能够降低下视频播放器的内存使用。关于VideoToolbox的介绍后面再补上[占坑专用]()。在开发过程中，参考了两个Demo：

1. [VTDemo](https://github.com/lileilei1119/VTDemo)

2. [-VideoToolboxDemo](https://github.com/adison/-VideoToolboxDemo)

这两个Demo中，基本思路和代码都是差不多。利用FFMPEG进行解析出流数据之后，利用VideoToolbox转换出画面输出。但是在开发过程中，我利用这两个Demo却一直找不到H.264的开始码。这里贴一下两个Demo寻找开始码的代码片段如下：

```objective-c
// -VideoTolbox 寻找H.264开始码

- (void) iOS8HWDecode
{
    // 1. get SPS,PPS form stream data, and create CMFormatDescription 和 VTDecompressionSession
    if (spsData == nil && ppsData == nil) {
        uint8_t *data = pCodecCtx -> extradata;
        int size = pCodecCtx -> extradata_size;
        NSString *tmp3 = [NSString new];
        for(int i = 0; i < size; i++) {
            NSString *str = [NSString stringWithFormat:@" %.2X",data[i]];
            tmp3 = [tmp3 stringByAppendingString:str];
        }

        int startCodeSPSIndex = 0;
        int startCodePPSIndex = 0;
        int spsLength = 0;
        int ppsLength = 0;

        for (int i = 0; i < size; i++) {
            if (i >= 3) {
                if (data[i] == 0x01 && data[i-1] == 0x00 && data[i-2] == 0x00 && data[i-3] == 0x00) {
                    if (startCodeSPSIndex == 0) {
                        startCodeSPSIndex = i;
                    }
                    if (i > startCodeSPSIndex) {
                        startCodePPSIndex = i;
                    }
                }
            }
        }

        spsLength = startCodePPSIndex - startCodeSPSIndex - 4;
        ppsLength = size - (startCodePPSIndex + 1);

        int nalu_type;
        nalu_type = ((uint8_t) data[startCodeSPSIndex + 1] & 0x1F);
//        NSLog(@"NALU with Type \"%@\" received.", naluTypesStrings[nalu_type]);
        if (nalu_type == 7) {
            spsData = [NSData dataWithBytes:&(data[startCodeSPSIndex + 1]) length: spsLength];
        }

        nalu_type = ((uint8_t) data[startCodePPSIndex + 1] & 0x1F);
//        NSLog(@"NALU with Type \"%@\" received.", naluTypesStrings[nalu_type]);
        if (nalu_type == 8) {
            ppsData = [NSData dataWithBytes:&(data[startCodePPSIndex + 1]) length: ppsLength];
        }

        // 2. create  CMFormatDescription
        if (spsData != nil && ppsData != nil) {
            const uint8_t* const parameterSetPointers[2] = { (const uint8_t*)[spsData bytes], (const uint8_t*)[ppsData bytes] };
            const size_t parameterSetSizes[2] = { [spsData length], [ppsData length] };
            status = CMVideoFormatDescriptionCreateFromH264ParameterSets(kCFAllocatorDefault, 2, parameterSetPointers, parameterSetSizes, 4, &videoFormatDescr);
//            NSLog(@"Found all data for CMVideoFormatDescription. Creation: %@.", (status == noErr) ? @"successfully." : @"failed.");
        }

        // 3. create VTDecompressionSession
        VTDecompressionOutputCallbackRecord callback;
        callback.decompressionOutputCallback = didDecompress;
        callback.decompressionOutputRefCon = (__bridge void *)self;
          NSDictionary *destinationImageBufferAttributes =[NSDictionary dictionaryWithObjectsAndKeys:[NSNumber numberWithBool:NO],(id)kCVPixelBufferOpenGLESCompatibilityKey,[NSNumber numberWithInt:kCVPixelFormatType_32BGRA],(id)kCVPixelBufferPixelFormatTypeKey,nil];
//        NSDictionary *destinationImageBufferAttributes =[NSDictionary dictionaryWithObjectsAndKeys:[NSNumber numberWithBool:NO],(id)kCVPixelBufferOpenGLESCompatibilityKey,nil];
//        NSDictionary *destinationImageBufferAttributes = [NSDictionary dictionaryWithObject: [NSNumber numberWithInt:kCVPixelFormatType_32BGRA] forKey: (id)kCVPixelBufferPixelFormatTypeKey];
        status = VTDecompressionSessionCreate(kCFAllocatorDefault, videoFormatDescr, NULL, (CFDictionaryRef)destinationImageBufferAttributes, &callback, &session);
//        status = VTDecompressionSessionCreate(kCFAllocatorDefault, videoFormatDescr, NULL, NULL, &callback, &session);
//        NSLog(@"Creating Video Decompression Session: %@.", (status == noErr) ? @"successfully." : @"failed.");


        int32_t timeSpan = 90000;
        CMSampleTimingInfo timingInfo;
        timingInfo.presentationTimeStamp = CMTimeMake(0, timeSpan);
        timingInfo.duration =  CMTimeMake(3000, timeSpan);
        timingInfo.decodeTimeStamp = kCMTimeInvalid;
    }

    int startCodeIndex = 0;
    for (int i = 0; i < 5; i++) {
        if (packet.data[i] == 0x01) {
            startCodeIndex = i;
            break;
        }
    }
    int nalu_type = ((uint8_t)packet.data[startCodeIndex + 1] & 0x1F);
//    NSLog(@"NALU with Type \"%@\" received.", naluTypesStrings[nalu_type]);

    if (nalu_type == 1 || nalu_type == 5) {
        // 4. get NALUnit payload into a CMBlockBuffer,
        CMBlockBufferRef videoBlock = NULL;
        status = CMBlockBufferCreateWithMemoryBlock(NULL, packet.data, packet.size, kCFAllocatorNull, NULL, 0, packet.size, 0, &videoBlock);
//        NSLog(@"BlockBufferCreation: %@", (status == kCMBlockBufferNoErr) ? @"successfully." : @"failed.");

        // 5.  making sure to replace the separator code with a 4 byte length code (the length of the NalUnit including the unit code)
        int reomveHeaderSize = packet.size - 4;
        const uint8_t sourceBytes[] = {(uint8_t)(reomveHeaderSize >> 24), (uint8_t)(reomveHeaderSize >> 16), (uint8_t)(reomveHeaderSize >> 8), (uint8_t)reomveHeaderSize};
        status = CMBlockBufferReplaceDataBytes(sourceBytes, videoBlock, 0, 4);
//        NSLog(@"BlockBufferReplace: %@", (status == kCMBlockBufferNoErr) ? @"successfully." : @"failed.");

        NSString *tmp3 = [NSString new];
        for(int i = 0; i < sizeof(sourceBytes); i++) {
            NSString *str = [NSString stringWithFormat:@" %.2X",sourceBytes[i]];
            tmp3 = [tmp3 stringByAppendingString:str];
        }
//        NSLog(@"size = %i , 16Byte = %@",reomveHeaderSize,tmp3);

        // 6. create a CMSampleBuffer.
        CMSampleBufferRef sbRef = NULL;
        const size_t sampleSizeArray[] = {packet.size};
//        status = CMSampleBufferCreate(kCFAllocatorDefault, videoBlock, true, NULL, NULL, videoFormatDescr, 1, 1, &timingInfo, 1, sampleSizeArray, &sbRef);
        status = CMSampleBufferCreate(kCFAllocatorDefault, videoBlock, true, NULL, NULL, videoFormatDescr, 1, 0, NULL, 1, sampleSizeArray, &sbRef);

//        NSLog(@"SampleBufferCreate: %@", (status == noErr) ? @"successfully." : @"failed.");

        // 7. use VTDecompressionSessionDecodeFrame
        VTDecodeFrameFlags flags = kVTDecodeFrame_EnableAsynchronousDecompression;
        VTDecodeInfoFlags flagOut;
        status = VTDecompressionSessionDecodeFrame(session, sbRef, flags, &sbRef, &flagOut);
//        NSLog(@"VTDecompressionSessionDecodeFrame: %@", (status == noErr) ? @"successfully." : @"failed.");
        CFRelease(sbRef);

        [self.delegate startDecodeData];
    }
}
```

```objc
// VTDemo 寻找H.264
-(void)loadFrame
{
    while (av_read_frame(pFormatCtx, &packet)>= 0) {
        if (packet.stream_index == streamNo) {
            NSLog(@"=========dddd=========");
            [_h264Decoder decodeFrame:packet.data withSize:packet.size];
        }
    }
}

-(void) decodeFrame:(uint8_t *)frame withSize:(uint32_t)frameSize
{
    OSStatus status;

    uint8_t *data = NULL;
    uint8_t *pps = NULL;
    uint8_t *sps = NULL;

    int startCodeIndex = 0;
    int secondStartCodeIndex = 0;
    int thirdStartCodeIndex = 0;

    long blockLength = 0;

    CMSampleBufferRef sampleBuffer = NULL;
    CMBlockBufferRef blockBuffer = NULL;

    int nalu_type = (frame[startCodeIndex + 4] & 0x1F);

    if (nalu_type != 7 && _formatDesc == NULL)
    {
        NSLog(@"Video error: Frame is not an I Frame and format description is null");
        return;
    }

    if (nalu_type == 7)
    {
        // 去掉起始头0x00 00 00 01   有的为0x00 00 01
        for (int i = startCodeIndex + 4; i < startCodeIndex + 44; i++)
        {
            if (frame[i] == 0x00 && frame[i+1] == 0x00 && frame[i+2] == 0x00 && frame[i+3] == 0x01)
            {
                secondStartCodeIndex = i;
                _spsSize = secondStartCodeIndex;
                break;
            }
        }

        nalu_type = (frame[secondStartCodeIndex + 4] & 0x1F);
    }

    if(nalu_type == 8)
    {
        for (int i = _spsSize + 4; i < _spsSize + 60; i++)
        {
            if (frame[i] == 0x00 && frame[i+1] == 0x00 && frame[i+2] == 0x00 && frame[i+3] == 0x01)
            {
                thirdStartCodeIndex = i;
                _ppsSize = thirdStartCodeIndex - _spsSize;
                break;
            }
        }

        sps = malloc(_spsSize - 4);
        pps = malloc(_ppsSize - 4);

        memcpy (sps, &frame[4], _spsSize-4);
        memcpy (pps, &frame[_spsSize+4], _ppsSize-4);

        uint8_t*  parameterSetPointers[2] = {sps, pps};
        size_t parameterSetSizes[2] = {_spsSize-4, _ppsSize-4};

        status = CMVideoFormatDescriptionCreateFromH264ParameterSets(kCFAllocatorDefault, 2,
                                                                     (const uint8_t *const*)parameterSetPointers,
                                                                     parameterSetSizes, 4,
                                                                     &_formatDesc);


        nalu_type = (frame[thirdStartCodeIndex + 4] & 0x1F);
    }

    if((status == noErr) && (_decompressionSession == NULL))
    {
        [self createDecompSession];
    }

    if(nalu_type == 5)
    {

        int offset = _spsSize + _ppsSize;
        blockLength = frameSize - offset;
        data = malloc(blockLength);
        data = memcpy(data, &frame[offset], blockLength);

        uint32_t dataLength32 = htonl (blockLength - 4);
        memcpy (data, &dataLength32, sizeof (uint32_t));


        status = CMBlockBufferCreateWithMemoryBlock(NULL, data,
                                                    blockLength,
                                                    kCFAllocatorNull, NULL,
                                                    0,
                                                    blockLength,
                                                    0, &blockBuffer);

        NSLog(@"\t\t BlockBufferCreation: \t %@", (status == kCMBlockBufferNoErr) ? @"successful!" : @"failed...");
    }

    if (nalu_type == 1)
    {
        blockLength = frameSize;
        data = malloc(blockLength);
        data = memcpy(data, &frame[0], blockLength);

        uint32_t dataLength32 = htonl (blockLength - 4);
        memcpy (data, &dataLength32, sizeof (uint32_t));

        status = CMBlockBufferCreateWithMemoryBlock(NULL, data,
                                                    blockLength,
                                                    kCFAllocatorNull, NULL,
                                                    0,
                                                    blockLength,
                                                    0, &blockBuffer);
    }

    if(status == noErr)
    {
        const size_t sampleSize = blockLength;
        status = CMSampleBufferCreate(kCFAllocatorDefault,
                                      blockBuffer, true, NULL, NULL,
                                      _formatDesc, 1, 0, NULL, 1,
                                      &sampleSize, &sampleBuffer);

        NSLog(@"\t\t SampleBufferCreate: \t %@", (status == noErr) ? @"successful!" : @"failed...");
    }

    if(status == noErr)
    {
        CFArrayRef attachments = CMSampleBufferGetSampleAttachmentsArray(sampleBuffer, YES);
        CFMutableDictionaryRef dict = (CFMutableDictionaryRef)CFArrayGetValueAtIndex(attachments, 0);
        CFDictionarySetValue(dict, kCMSampleAttachmentKey_DisplayImmediately, kCFBooleanTrue);

        [self render:sampleBuffer];
    }


    if (NULL != blockBuffer) {
        CFRelease(blockBuffer);
        blockBuffer = NULL;
    }

    [self relaseData:data];
    [self relaseData:pps];
    [self relaseData:sps];

    [self.delegate startDecodeData];
}
```

从代码段中我们能看到，思路都是一样的，但是，其中-VideoToolboxDemo寻找开始码使用的数据是`_pCodeCtx->data`,而VTDemo使用的数据却是`packet.data`。然而我试了这两种方法之后，都没能获取到正确的结果。花了很多时间之后，只能上Stack Overflow上提问：

[how-to-hardcode-a-mp4-stream-file-with-ios-videotoolbox-and-ffmpeg](http://stackoverflow.com/questions/37347432/how-to-hardcode-a-mp4-stream-file-with-ios-videotoolbox-and-ffmpeg/37351157#37351157)

最后才找到答案，解决了通过找H.264的开始码来寻找SPS和PPS数据的问题。

现在就将答案翻译下，希望对大家有点帮助。

----

首先，我们需要知道的一点就是没有一个统一标准的H.264基本码流格式。其规格文档中规定了Annex，尤其是Annex B这种非实际需求的格式。这种标准描述了视频编码到每个独立的packet，并且这些packet保存和传输的方式。

# Annex B

## Network Abstraction Layer Units

Network Abstraction Layer Units，简称为NALU，用于每个packet的解析和处理。每个NALU的首个字节的第3-7bit代表了NALU的类型（第0bit十关闭的，1-2bit表明NALU是否和另一个NALU相关）

NALU可以分为两大类VCL和non-VCL, 共19种类型

|Order   |Type                               |VCL or Non-VCL|
|--------|:---------------------------------:|:------------:|
|0       |Unspecified   |non-VCL       |
|1       |Coded slice of a non-IDR picture   |VCL           |
|2       |Coded slice data partition A   |VCL           |
|3       |Coded slice data partition B   |VCL           |
|4       |Coded slice data partition C   |VCL           |
|5       |Coded slice of an IDR picture   |VCL           |
|6       |Supplemental enhancement information (SEI)   |non-VCL           |
|7       |Sequence parameter set   |non-VCL           |
|8       |Picture parameter set   |non-VCL           |
|9       |Access unit delimiter   |non-VCL           |
|10      |End of sequence   |non-VCL           |
|11      |End of stream   |non-VCL           |
|12      |Filler data   |non-VCL           |
|13      |Sequence parameter set extension   |non-VCL           |
|14      |Prefix NAL unit   |non-VCL           |
|15      |Subset sequence parameter set   |non-VCL           |
|16      |Depth parameter set   |non-VCL           |
|17..18  |Reserved   |non-VCL           |
|19      |Coded slice of an auxiliary coded picture without partitioning   |non-VCL           |
|20      |Coded slice extension   |non-VCL           |
|21      |Coded slice extension for depth view components   |non-VCL           |
|22..23  |Reserved   |non-VCL           |
|24..31  |Unspecified   |non-VCL           |


Sequence Parameter Set(SPS). 这个non-VCL NALU包含了配置解码器的所需要的配置文件，分辨率，帧率等信息。

Picture Parameter Set(PPS)跟SPS类似，包含了熵编码模式，片段组，运动预测和解封滤波器的信息。

Instantaneous Decoder Refresh (IDR). 这个VCL NALU是一个自包含图像片。也就是IDR可以解码和播放，而不需要通过PPS和SPS。

Access Unit Delimiter(AUD)是可选的NALU用于在基础码流中划分帧。

### NALU Start Codes

NALU的内容中没有包含其size信息。因此简单地进行NALU连接并不能完成解析播放的目的，因为我们不知道NALU的在什么地方开始，在什么地方结束。

Annex B的规格文档通过在每个NALU前添加开始码来解决这个问题。开始码是2-3个0x00字节，加上一个0x01字节组成的。比方说0x000001或者是0x00000001.

除此以外，开始码还有一个4个字节的变种，适合在串联连接中进行传输。比方说，如果下一个bit是0，那么就是一个NALU的起始位置。这种变种通常只用在信令流中的随机接入点，比方说SPS PPS AUD和IDR等这些用在其他地方的，以便节省空间。

### Emulation Prevention Bytes

之所以需要开始码的原因是因为0x000000, 0x000001, 0x000002, 0x000003在non-RBSP NALU中是没办法使用的。所以在创建NALU的时候，需要注意避免这些值对开始码造成的影响。

在解码的时候，寻找并且忽略这些字节是比较关键的一步。因为这些字节可以出现在NALU的任何地方。通常较为方便的做法是假定这些字节已经被移除了。去掉这些字节之后的序列称之为Raw Byte Sequence Payload(RBSP)

e.g

0x0000 | 00 00 00 01 67 64 00 0A AC 72 84 44 26 84 00 00
0x0010 | 03 00 04 00 00 03 00 CA 3C 48 96 11 80 00 00 00
0x0020 | 01 68 E8 43 8F 13 21 30 00 00 01 65 88 81 00 05
0x0030 | 4E 7F 87 DF 61 A5 8B 95 EE A4 E9 38 B7 6A 30 6A
0x0040 | 71 B9 55 60 0B 76 2E B5 0E E4 80 59 27 B8 67 A9
0x0050 | 63 37 5E 82 20 55 FB E4 6A E9 37 35 72 E2 22 91
0x0060 | 9E 4D FF 60 86 CE 7E 42 B7 95 CE 2A E1 26 BE 87
0x0070 | 73 84 26 BA 16 36 F4 E6 9F 17 DA D8 64 75 54 B1
0x0080 | F3 45 0C 0B 3C 74 B3 9D BC EB 53 73 87 C3 0E 62
0x0090 | 47 48 62 CA 59 EB 86 3F 3A FA 86 B5 BF A8 6D 06
0x00A0 | 16 50 82 C4 CE 62 9E 4E E6 4C C7 30 3E DE A1 0B
0x00B0 | D8 83 0B B6 B8 28 BC A9 EB 77 43 FC 7A 17 94 85
0x00C0 | 21 CA 37 6B 30 95 B5 46 77 30 60 B7 12 D6 8C C5
0x00D0 | 54 85 29 D8 69 A9 6F 12 4E 71 DF E3 E2 B1 6B 6B
0x00E0 | BF 9F FB 2E 57 30 A9 69 76 C4 46 A2 DF FA 91 D9
0x00F0 | 50 74 55 1D 49 04 5A 1C D6 86 68 7C B6 61 48 6C
0x0100 | 96 E6 12 4C 27 AD BA C7 51 99 8E D0 F0 ED 8E F6
0x0110 | 65 79 79 A6 12 A1 95 DB C8 AE E3 B6 35 E6 8D BC
0x0120 | 48 A3 7F AF 4A 28 8A 53 E2 7E 68 08 9F 67 77 98
0x0130 | 52 DB 50 84 D6 5E 25 E1 4A 99 58 34 C7 11 D6 43
0x0140 | FF C4 FD 9A 44 16 D1 B2 FB 02 DB A1 89 69 34 C2
0x0150 | 32 55 98 F9 9B B2 31 3F 49 59 0C 06 8C DB A5 B2
0x0160 | 9D 7E 12 2F D0 87 94 44 E4 0A 76 EF 99 2D 91 18
0x0170 | 39 50 3B 29 3B F5 2C 97 73 48 91 83 B0 A6 F3 4B
0x0180 | 70 2F 1C 8F 3B 78 23 C6 AA 86 46 43 1D D7 2A 23
0x0190 | 5E 2C D9 48 0A F5 F5 2C D1 FB 3F F0 4B 78 37 E9
0x01A0 | 45 DD 72 CF 80 35 C3 95 07 F3 D9 06 E5 4A 58 76
0x01B0 | 03 6C 81 20 62 45 65 44 73 BC FE C1 9F 31 E5 DB
0x01C0 | 89 5C 6B 79 D8 68 90 D7 26 A8 A1 88 86 81 DC 9A
0x01D0 | 4F 40 A5 23 C7 DE BE 6F 76 AB 79 16 51 21 67 83
0x01E0 | 2E F3 D6 27 1A 42 C2 94 D1 5D 6C DB 4A 7A E2 CB
0x01F0 | 0B B0 68 0B BE 19 59 00 50 FC C0 BD 9D F5 F5 F8
0x0200 | A8 17 19 D6 B3 E9 74 BA 50 E5 2C 45 7B F9 93 EA
0x0210 | 5A F9 A9 30 B1 6F 5B 36 24 1E 8D 55 57 F4 CC 67
0x0220 | B2 65 6A A9 36 26 D0 06 B8 E2 E3 73 8B D1 C0 1C
0x0230 | 52 15 CA B5 AC 60 3E 36 42 F1 2C BD 99 77 AB A8
0x0240 | A9 A4 8E 9C 8B 84 DE 73 F0 91 29 97 AE DB AF D6
0x0250 | F8 5E 9B 86 B3 B3 03 B3 AC 75 6F A6 11 69 2F 3D
0x0260 | 3A CE FA 53 86 60 95 6C BB C5 4E F3

这是一个完整的包含了3个NALUs的AU。正如你能看到的，SPS（67）是以开始码打头开始。在SPS中，你可以看到两个Emulation Prevention bytes.然后PPS(68)也是以开始码开始的，最后一个开始码出现在IDR片段之前。这是一个完整的H.264码流。

Annex B一般用于Live和流格式的传输。这些格式会周期性地重复SPS和PPS

## AVCC

另外一种常用的保存H.264流的格式是AVCC。这种格式中，每个NALU前都是其长度。这种方法的话比较容易解析，但是你会丢失Annex B的字节对齐配置。可以使用1，2或者4个字节的长度进行编码。这个值存储于一个header对象中。该对象称为'extradata'或者'sqquence header'。基础格式如下：

bits    
8   version ( always 0x01 )
8   avc profile ( sps[0][1] )
8   avc compatibility ( sps[0][2] )
8   avc level ( sps[0][3] )
6   reserved ( all bits on )
2   NALULengthSizeMinusOne
3   reserved ( all bits on )
5   number of SPS NALUs (usually 1)
repeated once per SPS:
  16     SPS size
  variable   SPS NALU data
8   number of PPS NALUs (usually 1)
repeated once per PPS
  16    PPS size
  variable PPS NALU data

使用上面的例子来看的话，header对象数据如下：

0x0000 | 01 64 00 0A FF E1 00 19 67 64 00 0A AC 72 84 44
0x0010 | 26 84 00 00 03 00 04 00 00 03 00 CA 3C 48 96 11
0x0020 | 80 01 00 07 68 E8 43 8F 13 21 30

第一字节0x01表示的是版本, 0x64 00 0A则是SPS的前三位，而0xFF的前6bit是保留用，后两位表示NALULengthSizeMinusOne，0xE1前3bit为保留位，后5位是SPS NALUs的个数，这通常为1. 0x19代表的是SPS的长度，0x67~80就是SPS的数据内容，一直到0x07，代表PPS的长度，0x68开始到结尾就是PPS的内容。

你会发现SPS和PPS被存储在带外。这是因为已经从基础流数据中分离了出来。存储和传输这个数据是文件容器的工作。虽然在这里并没有使用开始码，但是emulation prevention bytes仍然被插入在PPS和SPS数据中。

我们可以看到上面有个NALULengthSizeMinusOne的变量。这个变量告诉我们有多少个字节用于存储NALU的长度。因此，如果它的值为0的话，则表示每个NALU前仅用一个字节来表示其长度。使用单个字节来保存size的话，NALU最大的字节是255字节。使用两个字节的话就是64k。对于我们的例子来说，3个字节是比较合适的，但是更常用的还是使用4个字节来进行表示

这种格式的好处就是，能够直接让解码器跳到我们需要的数据，较常用在MP4和MKV上面。

----

举个例子：

```
0x0000 | 00 00 02 41 65 88 81 00 05 4E 7F 87 DF 61 A5 8B
0x0010 | 95 EE A4 E9 38 B7 6A 30 6A 71 B9 55 60 0B 76 2E
0x0020 | B5 0E E4 80 59 27 B8 67 A9 63 37 5E 82 20 55 FB
0x0030 | E4 6A E9 37 35 72 E2 22 91 9E 4D FF 60 86 CE 7E
0x0040 | 42 B7 95 CE 2A E1 26 BE 87 73 84 26 BA 16 36 F4
0x0050 | E6 9F 17 DA D8 64 75 54 B1 F3 45 0C 0B 3C 74 B3
0x0060 | 9D BC EB 53 73 87 C3 0E 62 47 48 62 CA 59 EB 86
0x0070 | 3F 3A FA 86 B5 BF A8 6D 06 16 50 82 C4 CE 62 9E
0x0080 | 4E E6 4C C7 30 3E DE A1 0B D8 83 0B B6 B8 28 BC
0x0090 | A9 EB 77 43 FC 7A 17 94 85 21 CA 37 6B 30 95 B5
0x00A0 | 46 77 30 60 B7 12 D6 8C C5 54 85 29 D8 69 A9 6F
0x00B0 | 12 4E 71 DF E3 E2 B1 6B 6B BF 9F FB 2E 57 30 A9
0x00C0 | 69 76 C4 46 A2 DF FA 91 D9 50 74 55 1D 49 04 5A
0x00D0 | 1C D6 86 68 7C B6 61 48 6C 96 E6 12 4C 27 AD BA
0x00E0 | C7 51 99 8E D0 F0 ED 8E F6 65 79 79 A6 12 A1 95
0x00F0 | DB C8 AE E3 B6 35 E6 8D BC 48 A3 7F AF 4A 28 8A
0x0100 | 53 E2 7E 68 08 9F 67 77 98 52 DB 50 84 D6 5E 25
0x0110 | E1 4A 99 58 34 C7 11 D6 43 FF C4 FD 9A 44 16 D1
0x0120 | B2 FB 02 DB A1 89 69 34 C2 32 55 98 F9 9B B2 31
0x0130 | 3F 49 59 0C 06 8C DB A5 B2 9D 7E 12 2F D0 87 94
0x0140 | 44 E4 0A 76 EF 99 2D 91 18 39 50 3B 29 3B F5 2C
0x0150 | 97 73 48 91 83 B0 A6 F3 4B 70 2F 1C 8F 3B 78 23
0x0160 | C6 AA 86 46 43 1D D7 2A 23 5E 2C D9 48 0A F5 F5
0x0170 | 2C D1 FB 3F F0 4B 78 37 E9 45 DD 72 CF 80 35 C3
0x0180 | 95 07 F3 D9 06 E5 4A 58 76 03 6C 81 20 62 45 65
0x0190 | 44 73 BC FE C1 9F 31 E5 DB 89 5C 6B 79 D8 68 90
0x01A0 | D7 26 A8 A1 88 86 81 DC 9A 4F 40 A5 23 C7 DE BE
0x01B0 | 6F 76 AB 79 16 51 21 67 83 2E F3 D6 27 1A 42 C2
0x01C0 | 94 D1 5D 6C DB 4A 7A E2 CB 0B B0 68 0B BE 19 59
0x01D0 | 00 50 FC C0 BD 9D F5 F5 F8 A8 17 19 D6 B3 E9 74
0x01E0 | BA 50 E5 2C 45 7B F9 93 EA 5A F9 A9 30 B1 6F 5B
0x01F0 | 36 24 1E 8D 55 57 F4 CC 67 B2 65 6A A9 36 26 D0
0x0200 | 06 B8 E2 E3 73 8B D1 C0 1C 52 15 CA B5 AC 60 3E
0x0210 | 36 42 F1 2C BD 99 77 AB A8 A9 A4 8E 9C 8B 84 DE
0x0220 | 73 F0 91 29 97 AE DB AF D6 F8 5E 9B 86 B3 B3 03
0x0230 | B3 AC 75 6F A6 11 69 2F 3D 3A CE FA 53 86 60 95
0x0240 | 6C BB C5 4E F3
```

比方说上面这段data中，0x00 00 02 41只是用来表面NALU的长度，真正的SPS数据是从65开始的，所以在处理数据的时候，需要移除掉前四位

```objc
int startIndex = 4;
  int nalu_type = ((uint8_t)frame[startIndex] & 0x1F);
  if (nalu_type == 1) {
    CMBlockBufferRef videoBlock = NULL;
    _status = CMBlockBufferCreateWithMemoryBlock(NULL, frame, size, kCFAllocatorNull, NULL, 0, size, 0, &videoBlock);
    //        NSLog(@"BlockBufferCreation: %@", (status == kCMBlockBufferNoErr) ? @"successfully." : @"failed.");

    // 5.  making sure to replace the separator code with a 4 byte length code (the length of the NalUnit including the unit code)
    int reomveHeaderSize = size - 4;
    const uint8_t sourceBytes[] = {(uint8_t)(reomveHeaderSize >> 24), (uint8_t)(reomveHeaderSize >> 16), (uint8_t)(reomveHeaderSize >> 8), (uint8_t)reomveHeaderSize};
    _status = CMBlockBufferReplaceDataBytes(sourceBytes, videoBlock, 0, 4);

    ...
  }
```
