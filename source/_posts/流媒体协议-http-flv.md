---
title: 流媒体-http_flv"
date: 2023-05-14 21:16:04
categories:
- 流媒体
- http-flv
tags:
- 流媒体
---

# FLV是什么
FLV 是FLASH VIDEO的简称，FLV流媒体格式是随着Flash MX的推出发展而来的视频格式。由于它形成的文件极小、加载速度极快，使得网络观看视频文件成为可能，它的出现有效地解决了视频文件导入Flash后，使导出的SWF文件体积庞大，不能在网络上很好的使用等问题。

# FLV文件格式
FLV 由 FLV header 跟 FLV file body 两部分组成，而 FLV file body 又由多个 FLV tag组成，FLV tag由 tag header + tag body组成。
- FLV 文件头和文件体 (E.2, E.3)
从整个文件上看，FLV = FLV File Header + FLV File Body。

![](https://pic4.zhimg.com/80/v2-6fe937f1958d221c0ab0dfededcb6f5f_1440w.webp)
## FLV Tag
一般FLV tag 分为三类
+ Video Tag 0x08：audio data
+ Audio Tag 0x09：video data
+ Script Tag 0x12：script data

- FLV Tag (E.4)
![](https://pic1.zhimg.com/80/v2-bf301ba2d87829798cbf0ecde7c4063c_1440w.webp)

Timestamp 和 TimestampExtended 组成了这个 TAG 包数据的 PTS 信息，PTS = Timestamp | TimestampExtended << 24。

1. AudioTag (E.4.2)
由于 AAC 编码的特殊性，这里着重说明了 AAC 编码的 Tag 格式。
![](https://pic4.zhimg.com/80/v2-02f4db5905d43348d0e8a8b0747abaa7_1440w.webp)

AudioTagHeader 的第一个字节，也就是接跟着 StreamID 的 1 个字节包含了音频类型，采样率等的基本信息。
![](https://pic3.zhimg.com/80/v2-dd4697d2f7826195b739b928023d64ca_1440w.webp)

AudioTagHeader 之后跟着的就是 AUDIODATA 部分了。但是，这里有个特例，如果音频格式(SoundFormat)是 AAC，AudioTagHeader 中会多出 1 个字节的数据 AACPacketType，这个字段来表示 AACAUDIODATA 的类型：0 = AAC sequence header，1 = AAC raw。
AudioSpecificConfig 结构描述非常复杂，在标准文档中是用伪代码描述的，这里先假定要编码的音频格式，做一下简化。
音频编码为：AAC-LC，音频采样率为 44100。
![](https://pic1.zhimg.com/80/v2-f38b7dfc7b6731bdbf1861a1fa2e6bd8_1440w.webp)

在 FLV 的文件中，一般情况下 AAC sequence header 这种包只出现1次，而且是第一个 audio tag，为什么需要这种 tag，因为在做 FLV demux 的时候，如果是 AAC 的音频，需要在每帧 AAC ES 流前边添加 7 个字节 ADST 头，ADST 是解码器通用的格式，也就是说 AAC 的纯 ES 流要打包成 ADST 格式的 AAC 文件，解码器才能正常播放。就是在打包 ADST 的时候，需要 samplingFrequencyIndex 这个信息，samplingFrequencyIndex 最准确的信息是在 AudioSpecificConfig 中，这样，你就完全可以把 FLV 文件中的音频信息及数据提取出来，送给音频解码器正常播放了。

2. VideoTag (E.4.3)
Video类型表明Data中存储的是视频数据，由 Video Tag Header（5个字节）和 Video Data组成。视频的编码类型可以是H264、H265等等
![](https://pic3.zhimg.com/80/v2-7e25d86426a456fd23f88d4c0d68a7da_1440w.webp)

3. SCRIPTDATA (E.4.4)
Script Data Tags通常用来存放跟FLV中音视频相关的元数据信息（onMetaData），比如时长、长度、宽度等。它的定义相对复杂些，采用AMF（Action Message Format）封装了一系列数据类型，比如字符串、数值、数组等。
![](https://pic1.zhimg.com/80/v2-ca4c96b4c8bb091fdd90bcf5a2c26d28_1440w.webp)

3.1 onMetadata (E.5)
FLV metadata object 保存在 SCRIPTDATA 中, 叫 onMetaData。不同的软件生成的 FLV 的 properties 不同。
![](https://pic2.zhimg.com/80/v2-940fe48c3c7de8dc93a7aacfc1625dfd_1440w.webp)

keyframes 索引信息
官方的文档中并没有对 keyframes index 做描述，但是，flv 的这种结构每个 tag 又不像 TS 有同步头，如果没有 keyframes index 的话，需要按顺序读取每一个tag, seek 及快进快退的效果会非常差。后来在做 flv 文件合成的时候,发现网上有的 flv 文件将 keyframes 信息隐藏在 Script Tag 中。

keyframes 几乎是一个非官方的标准, 也就是民间标准。两个常用的操作 metadata 的工具是 flvtool2 和 FLVMDI，都是把 keyframes 作为一个默认的元信息项目。在 FLVMDI 的主页上有描述:  

keyframes: (Object) This object is added only if you specify the /k switch. 'keyframes' is known to FLVMDI and if /k switch is not specified, 'keyframes' object will be deleted. 'keyframes' object has 2 arrays: 'filepositions' and 'times'. Both arrays have the same number of elements, which is equal to the number of key frames in the FLV. Values in times array are in 'seconds'. Each correspond to the timestamp of the n'th key frame. Values in filepositions array are in 'bytes'. Each correspond to the fileposition of the nth key frame video tag (which starts with byte tag type 9)  

也就是说 keyframes 中包含着 2 个内容 “filepositions” 和 “times”分别指的是关键帧的文件位置和关键帧的 PTS。通过 keyframes 可以建立起自己的 Index，然后在 seek 和快进快退的操作中，快速有效地跳转到你想要找的关键帧位置进行处理。

## FLV格式整体图
![](https://pic3.zhimg.com/80/v2-3e5438e5997ec9e39403762fc07e2fce_1440w.webp)

# HTTP-FLV
回到最开始的介绍，HTTP-FLV，即将音视频数据封装成 FLV，然后通过 HTTP 协议传输给客户端。
HLS 其实是一个 “文本协议”，而并非流媒体协议。那么，什么样的协议才能称之为流媒体协议呢？
流(stream)： 数据在网络上按时间先后次序传输和播放的连续音/视频数据流。之所以可以按照顺序传输和播放连续是因为在类似 RTMP、FLV 协议中，每一个音视频数据都被封装成了包含时间戳信息头的数据包。而当播放器拿到这些数据包解包的时候能够根据时间戳信息把这些音视频数据和之前到达的音视频数据连续起来播放。MP4、MKV 等等类似这种封装，必须拿到完整的音视频文件才能播放，因为里面的单个音视频数据块不带有时间戳信息，播放器不能将这些没有时间戳信息数据块连续起来，所以就不能实时的解码播放。

## 延迟分析
理论上（除去网络延迟外），FLV 可以做到仅仅一个音视频 tag 的延迟。
相比 RTMP 的优点：
- 可以在一定程度上避免防火墙的干扰 （例如, 有的机房只允许 80 端口通过）；
- 可以很好的兼容 HTTP 302 跳转，做到灵活调度；
- 可以使用 HTTPS 做加密通道;
- 很好的支持移动端(Android，IOS);



参考文章：  
- [直播协议 HTTP-FLV 详解](https://zhuanlan.zhihu.com/p/28722048)
- [【流媒体协议】图解 FLV 协议 快速入门](https://zhuanlan.zhihu.com/p/495705490)


