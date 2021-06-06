# 基础知识

[原文地址](https://mp.weixin.qq.com/s/DsoEYydjmoWiEruZYQKCgQ)

## 视频相关

### 色彩空间

* RGB
* YUV：Y, Cr, Cb
* HSV/HSB
* HSI/HSL
* Lab
* CMY

{% hint style="info" %}
Q: YUV有什么优势?

A: 人眼对亮度比较敏感, 对色度不敏感, 所有可以减少UV分量, 压缩视频体积
{% endhint %}

### 视频编码

* H26X\(1/2/3/4/5\)系列: ITU\(international Telecommunication Union\)国际电传视讯联盟主导
* MPEG\(1/2/3/4\)系列: MPEG\(Moving Picutre Experts Group\)主导

## 音频

{% hint style="info" %}
声波的三要素：**频率**，**振幅**，**波形**。

频率：音阶的高低。

振幅：响度。

波形：音色。
{% endhint %}



脉冲编码调制\(PCM\)是原始数据

采集PCM\(Pulse Code Modulation\)的步骤

> 模拟信号 -&gt; 采样 -&gt; 量化 -&gt; 编码 -&gt; 数字信号
>
> 奈奎斯特采样定理：**为了不失真地恢复模拟信号，采样频率应该不小于模拟信号频谱中最高频率的2倍。**

采样率: 人耳能听到的最高频率为20kHz，所以为了满足人耳的听觉要求，采样率至少为40kHz，通常为44.1kHz，更高的通常为48kHz。

采样位数: 通常有8位、16位、32位。

_**PCM数据三要素：量化格式\(sampleFormat\), 采样率\(sampleRate\), 声道数\(channel\)。**_

关于FPS, BPS

* fps: frames per second
* bps: bits per second

### 音频编码

* WAV: pcm数据格式前面加上**44字节头**
* MP3： **LAME编码**
* FLAC
* APE
* WMA
* AAC
* Ogg: 有潜力的新编码格式， 免费。

### AAC格式

* ADIF\(Audio Data Interchange Format\): 音频数据交换格式, 这种格式的特征是可以确定的找到这个音频数据的开始，不需进行在音频数据流中间开始的解码，即它的解码必须在明确定义的开始处进行。这种格式常用在磁盘文件中。
* ADTS\(Audio data transport Stream\): 音频数据传输流。这种格式的特征是它是一个有同步字的比特流，解码可以在这个流中任何位置开始。它的特征类似于mp3数据流格式。

> ADTS可以在任意帧解码，它每一帧都有头信息。ADIF只有一个统一的头，所以必须得到所有的数据后解码。且这两种的header的格式也是不同的，目前一般编码后的都是ADTS格式的音频流。

### ADTS HEADER分析

**Adts\_fixed\_header**

|  | NO. of bits | 解释 |
| :--- | :--- | :--- |
| syncword | 12 | 同步头: all bits must be 1 |
| ID | 1 | 0: MPEG-4, 1: MPEG-2 |
| Layer | 2 | always: '00' |
| **protection\_absent** | 1 | 表示是否误码校验  Warning, set to 1 if there no CRC and 0 if there is CRC |
| **profile** | 2 | 0: Main profile, 1: Low Complexity profile\(LC\) 2: Scalable Samping Rate profile\(SSR\), 3: reserved |
| **sampling\_frequency\_index** | 4 | 表示使用的采样率下标 |
| **channel\_configuration** | 3 | 表示声道数，比如2表示立体声双声道 |
| original\_copy | 1 |  |
| home | 1 |  |

**Adts\_variable\_header**

| copyright\_indentification\_bit | 1 |  |
| :--- | :--- | :--- |
| copyright\_indentification\_start | 1 |  |
| **aac\_frame\_length** | 13 | ADTS帧的长度: ADTS头 + AAC 原始流 |
| adts\_buffer\_fullness |  |  |
|  |  |  |

| samplingFrequencyIndex | Value |
| :--- | :--- |
| 0x0 | 96000 |
| 0x1 | 88200 |
| 0x2 | 64000 |
| 0x3 | 48000 |
| 0x4 | 44100 |
| 0x5 | 32000 |
| 0x6 | 24000 |
| 0x7 | 22050 |
| 0x8 | 16000 |
| 0x9 | 12000 |
| 0xa | 11025 |
| 0xb | 8000 |
| 0xc | 7350 |
| 0xd | reserved |
| 0xe | reserved |

## 音视频容器

* MP4
* rmvb
* avi
* mov
* mkv

### IDR帧 & I帧 & GOP的关系

I帧是在最早出现的概念.

**IDR一定是I帧, 但是有时候我们不一定需要每个GOP的I帧都是IDR\(比如开放的GOP的I帧就不是IDR帧\), 一个GOP只可能有一个I帧.**

 That series of frames can be a GOP, a Group of Picture, **a video is composed by a series of GOPs, each GOP starting with an I-frame.**

When frames from a GOP reference frames from another GOP it’s called Open GOP. If not, it is called a Closed GOP.

 **IDR-frame ensures the following frames will not reference any frame before the IDR**.



编码器

> 编码阶段\(DTS\): I帧之后 一定是P帧

B帧大部分保存的是运动矢量信息

\*\*\*\*



