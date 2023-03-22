---
title: opus编解码相关介绍
date: 2023-03-21 21:04:00
tags: opus
category: codec
---

本文主要讲解在实际开发中遇到的关于Opus的编解码问题

## SDP中OPUS介绍

在 SDP 中，任何其它采样率都不起作用，在 rtp 传输时默认采用 48000, 可以在 [rfc7587 Section 4.1](https://www.rfc-editor.org/rfc/rfc7587) 中查看
如下所示：
```
Opus supports 5 different audio bandwidths, which can be adjusted
during a stream.  The RTP timestamp is incremented with a 48000 Hz
clock rate for all modes of Opus and all sampling rates.  The unit
for the timestamp is samples per single (mono) channel.  The RTP
timestamp corresponds to the sample time of the first encoded sample
in the encoded frame.  For data encoded with sampling rates other
than 48000 Hz, the sampling rate has to be adjusted to 48000 Hz.
```

如果想要降低带宽，那么可以使用 `maxplaybackrate` 和 `maxaveragebitrate `，标准文档如下所示
```
3.1.1.  Recommended Bitrate

   For a frame size of 20 ms, these are the bitrate "sweet spots" for  Opus in various configurations:

   o  8-12 kbit/s for NB speech,
   o  16-20 kbit/s for WB speech,
   o  28-40 kbit/s for FB speech,
   o  48-64 kbit/s for FB mono music, and
   o  64-128 kbit/s for FB stereo music.
```

例如：如果想要使用 8000HZ 的 OPUS，那么 SDP 应该如下所示：
```
   m=audio 54312 RTP/AVP 101
   a=rtpmap:101 opus/48000/2
   a=fmtp:101 maxplaybackrate=8000; sprop-maxcapturerate=8000; maxaveragebitrate=12000
```

## WebRTC 中非对称采样率工作

WebRTC 支持通信双方以非对称采样率工作。比如：A和B通话，A的采样率为48000，B的采样率为8000。如下所示

```
A-->B OfferSDP
a=rtpmap:111 opus/48000/2
a=rtcp-fb:111 transport-cc
a=fmtp:111 minptime=10;useinbandfec=1;maxplaybackrate=16000
```

```
B-->A AnswerSDP
a=rtpmap:111 opus/48000/2
a=rtcp-fb:111 transport-cc
a=fmtp:111 minptime=10;useinbandfec=1;maxplaybackrate=48000
```
最终，A 采样率为48000，B 采用率为 16000

## WebRTC 中声道数与编码器的关系

当声道数 2 时，OPUS 编码内部将使用 celt ,代码如下：
```cpp
///FIXME 暂时未查阅到
config.application = config.num_channels == 1 ? AudioEncoderOpus::kVoip : AudioEncoderOpus::kAudio;
```

## OPUS编解码

OPUS代码的时候使用 [opus](https://github.com/xiph/opus) 进行编解码，可以使用 `opus/doc/trivial_example.c` 进行测试



## 引用
[Linphone opus codec sampling rate](https://stackoverflow.com/questions/60580526/linphone-opus-codec-sampling-rate)
